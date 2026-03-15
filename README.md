# TaskBoard - N-Tier Architecture with Redis Caching and Load Balancing

## รายละเอียดนักศึกษา
- **ชื่อ-นามสกุล:** [ชื่อนักศึกษาของคุณ]
- **รหัสนักศึกษา:** [รหัสนักศึกษาของคุณ]
- **วิชา:** ENGCE301 Software Architecture
- **ภาคเรียน:** [ภาคเรียน]
- **ปีการศึกษา:** [ปีการศึกษา]

## คำอธิบายโปรเจกต์
โปรเจกต์นี้เป็นการพัฒนาแอปพลิเคชัน TaskBoard โดยใช้สถาปัตยกรรมแบบ N-Tier ร่วมกับ Redis สำหรับการ caching และ Load Balancing เพื่อรองรับผู้ใช้จำนวนมาก โดยใช้ Docker Containers เพื่อแยกแต่ละส่วนของระบบอย่างชัดเจน

วัตถุประสงค์ของโปรเจกต์:
- ศึกษาการออกแบบระบบ N-Tier Architecture
- เรียนรู้การใช้งาน Redis สำหรับการ caching
- เข้าใจหลักการ Load Balancing และ Horizontal Scaling
- ประยุกต์ใช้ Docker Containers ในระบบจริง

## คุณสมบัติของระบบ
- จัดการงาน (Task Management) ด้วย Kanban Board
- แยกงานเป็น 3 สถานะ: TODO, IN PROGRESS, DONE
- ระบบ Caching ด้วย Redis เพื่อเพิ่มประสิทธิภาพ
- Load Balancing ด้วย Nginx
- Horizontal Scaling สำหรับ Application Servers
- ฐานข้อมูล PostgreSQL สำหรับเก็บข้อมูลถาวร
- API RESTful สำหรับการสื่อสาร
- Frontend ด้วย HTML/CSS/JavaScript

## เทคโนโลยีที่ใช้
- **Backend:** Node.js, Express.js
- **Database:** PostgreSQL
- **Cache:** Redis
- **Load Balancer:** Nginx
- **Containerization:** Docker, Docker Compose
- **Frontend:** HTML5, CSS3, JavaScript (Vanilla)

## สถาปัตยกรรมของระบบ
ระบบนี้ใช้สถาปัตยกรรมแบบ 4-Tier Architecture:

| Tier | Component | Description |
|------|-----------|-------------|
| Tier 1 | Nginx | Web Server และ Load Balancer |
| Tier 2 | Node.js API | Application Server (หลาย instances) |
| Tier 3a | Redis | In-Memory Cache |
| Tier 3b | PostgreSQL | Persistent Database |

```
Client (Browser)
      │
      ▼
┌────────────────────┐
│  Nginx LoadBalancer │
└──────────┬─────────┘
           │
     ┌─────┼─────┬─────┐
     ▼     ▼     ▼
   App1   App2   App3
     │      │      │
     └──────┼──────┘
            │
      ┌─────┴─────┐
      ▼           ▼
    Redis      PostgreSQL
   (Cache)      (Database)
```

## การติดตั้งและการรันระบบ

### ข้อกำหนดเบื้องต้น
- Docker และ Docker Compose ติดตั้งในระบบ
- Node.js (สำหรับการพัฒนา local)
- Git

### ขั้นตอนการติดตั้ง
1. Clone โปรเจกต์
   ```
   git clone [repository-url]
   cd engce301-week6_lab_ntier_redis-main
   ```

2. สร้างและรัน containers
   ```
   docker compose up -d
   ```

3. Scale application instances (optional)
   ```
   docker compose up -d --scale app=3
   ```

4. เข้าถึงแอปพลิเคชัน
   - Frontend: http://localhost
   - API Health Check: http://localhost/api/health

### คำสั่งที่มีประโยชน์
- ดูสถานะ containers: `docker compose ps`
- ดู logs: `docker compose logs -f [service-name]`
- หยุดระบบ: `docker compose down`
- ลบ volumes: `docker compose down -v`

## API Documentation

### Base URL
```
http://localhost/api
```

### Endpoints

#### Health Check
- **GET** `/health`
- ตรวจสอบสถานะของระบบและสถิติ cache

#### Tasks
- **GET** `/tasks` - ดูรายการงานทั้งหมด
- **GET** `/tasks/stats` - ดูสถิติของงาน
- **GET** `/tasks/:id` - ดูรายละเอียดงานตาม ID
- **POST** `/tasks` - สร้างงานใหม่
  - Body: `{ "title": "string", "priority": "LOW|MEDIUM|HIGH" }`
- **PUT** `/tasks/:id` - อัปเดตงาน
  - Body: `{ "title": "string", "status": "TODO|IN_PROGRESS|DONE", "priority": "LOW|MEDIUM|HIGH" }`
- **DELETE** `/tasks/:id` - ลบงาน

## กลยุทธ์การ Caching

ระบบใช้ **Cache-Aside Pattern** ในการจัดการ cache:

### การอ่านข้อมูล (Read Operation)
1. ตรวจสอบข้อมูลใน Redis ด้วย key `tasks:all`
2. ถ้าพบข้อมูล (Cache HIT) ส่งกลับทันที
3. ถ้าไม่พบ (Cache MISS) Query จาก PostgreSQL
4. บันทึกข้อมูลลง Redis พร้อม TTL 60 วินาที

### การเขียนข้อมูล (Write Operation)
1. บันทึกข้อมูลลง PostgreSQL
2. ลบ cache key `tasks:all` เพื่อให้ข้อมูลเป็นปัจจุบัน

## การทดสอบระบบ

### การทดสอบ Load Balancing
1. Scale application เป็น 3 instances
   ```
   docker compose up -d --scale app=3
   ```
2. เข้าถึง `/api/health` หลายครั้ง
3. สังเกต Instance ID ที่เปลี่ยนไปแสดงว่า Load Balancing ทำงาน

### การทดสอบ Caching
1. ดูสถิติ cache ที่ `/api/health`
2. สังเกต hit rate และจำนวน hits/misses

### การทดสอบ Scalability
1. เพิ่มจำนวน app instances
2. ทดสอบการทำงานของระบบภายใต้โหลดสูง

## สรุปและบทเรียน

### จุดเด่นของระบบ
- **ประสิทธิภาพ:** Redis caching ลดเวลาตอบสนองจาก ~50ms เหลือ ~2ms
- **ความสามารถในการขยาย:** สามารถเพิ่ม instances ได้ง่ายด้วย Docker
- **ความพร้อมใช้งาน:** ระบบยังทำงานได้แม้ instance หนึ่งจะล่ม
- **การแยกส่วน:** แต่ละ tier แยกกันชัดเจน อำนวยความสะดวกในการบำรุงรักษา

### ความท้าทาย
- ความซับซ้อนในการจัดการหลาย containers
- การจัดการ cache invalidation
- การ debug ในสภาพแวดล้อม containerized

### บทเรียนสำคัญ
- ความแตกต่างระหว่าง Layer (logical) และ Tier (physical)
- ความสำคัญของ TTL ใน cache เพื่อป้องกัน stale data
- แอปพลิเคชันควรเป็น stateless เพื่อรองรับ load balancing
- การออกแบบระบบต้องคำนึงถึง trade-offs ระหว่าง performance, complexity, และ maintainability

โปรเจกต์นี้ช่วยให้เข้าใจหลักการออกแบบระบบที่รองรับผู้ใช้จำนวนมาก และเป็นพื้นฐานที่ดีสำหรับการพัฒนาระบบขนาดใหญ่ในอนาคต.
