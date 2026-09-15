# PHASE 1 — AUTHENTICATION, USER ACCESS & ROLE CONTROL

## Objective

พัฒนาและตรวจสอบระบบเข้าใช้งานของ Application ก่อนทำ Phase อื่น

Application นี้ใช้ Google Firebase / Firestore เป็นระบบเดิม

Phase นี้ให้เน้นเฉพาะ:

- User Access
- Email + Password Login
- First-Time Password Setup
- Forgot Password
- Reset Password
- User Status
- Role-Based Access Control
- Security Rules
- Existing Data Preservation

**ยังไม่ต้องแก้ Recipe, Cost, Master Items หรือระบบ CSV ใน Phase นี้**

---

# 1. CRITICAL RULES

ก่อนแก้ไขใด ๆ:

1. Inspect existing Firebase project configuration
2. Inspect existing Firestore user-related collections
3. Inspect current login implementation
4. Inspect current security rules
5. Inspect existing user records
6. Preserve all valid existing user data

ห้าม:

- Reset Firebase Project
- Delete existing users
- Delete existing Firestore documents
- Seed demo users
- Create sample accounts
- Replace production data
- สร้าง Google Login
- สร้าง Social Login
- สร้าง Public Sign Up

หากระบบเดิมมี Authentication อยู่แล้ว ให้แก้เฉพาะส่วนที่จำเป็นและรักษาข้อมูลเดิมให้มากที่สุด

---

# 2. AUTHENTICATION MODEL

ระบบ Login ใช้:

**Email + Password**

เท่านั้น

ไม่ใช้:

- Google Sign-In
- Google OAuth
- Facebook Login
- LINE Login
- Apple Login
- Social Login ทุกประเภท

หน้า Login ต้องไม่มี Social Login button

---

# 3. ACCOUNT CREATION CONCEPT

ระบบนี้เป็น Internal Application

ผู้ใช้ทั่วไป **ห้ามสมัคร Account เอง**

Admin เป็นผู้กำหนดว่า Email ใดสามารถเข้าใช้งานระบบได้

Admin กำหนด:

- Email
- Display Name
- Role
- Active Status

Admin **ไม่ต้องกำหนด Password**

Password ต้องถูกตั้งโดยเจ้าของ Email เอง

---

# 4. FIRST-TIME ACCESS FLOW

ต้องสร้าง Flow สำหรับผู้ใช้ใหม่ดังนี้:

### Step 1

Admin เพิ่มผู้ใช้ในระบบ

ข้อมูลอย่างน้อย:

```text
Email
Display Name
Role
Status = Active
```

ตัวอย่าง:

```text
email: employee@company.com
displayName: Somchai
role: SALES
status: ACTIVE
```

---

### Step 2

ผู้ใช้เปิดหน้า:

**First-Time Access / Set Password**

---

### Step 3

ผู้ใช้กรอก Email

---

### Step 4

ระบบตรวจว่า Email นี้อยู่ใน Approved User List หรือไม่

ถ้าไม่พบ:

```text
This email is not authorized to access this application.
```

ห้ามให้สร้าง Password

ห้ามสร้าง User ใหม่อัตโนมัติ

---

### Step 5

ถ้า Email ได้รับอนุญาต และยังไม่เคยตั้ง Password:

ให้ผู้ใช้ตั้ง:

```text
New Password
Confirm Password
```

---

### Step 6

หลังตั้ง Password สำเร็จ:

Account พร้อม Login

หลังจากนั้นผู้ใช้ใช้:

```text
Email
Password
```

ในการเข้า Application

---

# 5. IMPORTANT SECURITY RULE FOR FIRST-TIME SETUP

ห้ามใช้เพียงการตรวจ Email บน Frontend แล้วอนุญาตให้สร้าง Password ทันที

ต้องมี Server-side / Firebase-side validation ตาม Architecture ที่เหมาะสม

เป้าหมายคือ:

คนที่รู้ Email ของพนักงานคนอื่น ต้องไม่สามารถ Claim Account ได้ง่าย ๆ

หาก Firebase Architecture ต้องใช้ Invitation / Secure Token / Password Setup Link เพื่อให้ปลอดภัย ให้ใช้แนวทางนั้นแทนการทำ Flow ที่ไม่ปลอดภัย

แต่ User Experience ยังคงต้องเป็น:

**Admin ระบุ Email → เจ้าของ Email ตั้ง Password เอง**

Admin ห้ามรู้ Password

---

# 6. LOGIN FLOW

หน้า Login แสดง:

```text
Email
Password

[เข้าสู่ระบบ]

ลืมรหัสผ่าน
```

สามารถมี:

- Show Password
- Hide Password

ไม่ต้องมี:

- Sign Up
- Register
- Continue with Google
- Continue with Facebook
- Continue with LINE

---

# 7. LOGIN VALIDATION

เมื่อ Login:

ระบบต้องตรวจ:

1. Email ถูกต้อง
2. Password ถูกต้อง
3. User อยู่ใน Approved User List
4. User Status = ACTIVE
5. User Role ถูกต้อง

หาก User ถูก Disable:

ต้อง Login ไม่ได้ทันที

แม้ Authentication Credential จะยังมีอยู่ก็ตาม

---

# 8. FORGOT PASSWORD

หน้า Login มี:

**Forgot Password / ลืมรหัสผ่าน**

Flow:

```text
Login
↓
Forgot Password
↓
Enter Email
↓
Send Reset Email
↓
User receives Email
↓
Set New Password
↓
Login again
```

ใช้ Email ที่ลงทะเบียนไว้เท่านั้น

---

# 9. PASSWORD RESET

User ต้องสามารถ Reset Password ผ่าน Email ได้ด้วยตัวเอง

Admin ไม่ควร:

- เห็น Password เดิม
- ดู Password ปัจจุบัน
- ส่ง Password ปัจจุบันให้ User

Admin สามารถช่วยได้โดย:

- Confirm ว่า Email ยัง Active
- Trigger Password Reset Process หาก Architecture รองรับ

Password ใหม่ต้องถูกกำหนดโดย User เอง

---

# 10. PASSWORD RULE

Password ขั้นต่ำ:

```text
Minimum 8 characters
```

รองรับ:

- Letters
- Numbers
- Special Characters

ไม่ต้องสร้าง Policy ที่ซับซ้อนจนใช้งานลำบากโดยไม่จำเป็น

Password ต้องไม่ถูกเก็บเป็น Plain Text ใน Firestore

---

# 11. USER ROLES

ระบบมี 3 Roles:

```text
ADMIN
TRAINER
SALES
```

---

# 12. ADMIN PERMISSIONS

ADMIN สามารถ:

- ดู User List
- เพิ่ม Email ที่อนุญาต
- กำหนด Display Name
- กำหนด Role
- Change Role
- Disable User
- Enable User
- ดู Last Login
- Trigger Reset Password Process หากเหมาะสม

Admin ไม่สามารถ:

- ดู Password
- Copy Password
- Export Password

---

# 13. TRAINER PERMISSIONS

TRAINER ในภาพรวมของ Application สามารถ:

- Read Recipe
- Create/Edit Recipe
- View Recipe History
- Search
- Export
- Support Guides
- Team Notes

แต่ Phase 1 ยังไม่ต้อง Implement Feature เหล่านี้ใหม่

Phase นี้ให้ Implement เฉพาะ Permission Framework เพื่อรองรับ Phase ถัดไป

TRAINER ห้าม:

- User Management
- Price Management
- Change another user's role

---

# 14. SALES PERMISSIONS

SALES ในภาพรวมของ Applicationเป็น Read / Share oriented user

อนาคตสามารถ:

- Search Recipes
- View Recipes
- Export
- Support Guides
- Team Notes

ห้าม:

- Edit Recipe
- Edit Price
- User Management
- Import Master Data

Phase 1 ให้สร้าง Permission Framework รองรับไว้ก่อน

---

# 15. AUTHENTICATION VS AUTHORIZATION

ต้องแยกชัดเจน

## Authentication

ตรวจว่า:

```text
Who is this user?
```

## Authorization

ตรวจว่า:

```text
What is this user allowed to do?
```

การ Login สำเร็จไม่ได้หมายความว่า User มีสิทธิ์ทำทุกอย่าง

---

# 16. BACKEND / FIREBASE SECURITY

ห้ามใช้ Frontend UI เป็น Security Layer เพียงอย่างเดียว

ตัวอย่าง:

ถึง SALES จะเปิด Developer Tools หรือพยายามเรียก Write Operation โดยตรง

ระบบต้อง Reject ตาม Role

Permission ต้องตรวจใน Layer ที่เชื่อถือได้ เช่น:

- Firestore Security Rules
- Secure Backend
- Cloud Functions
- Server-side validation

ขึ้นอยู่กับ Architecture เดิมของ Project

---

# 17. FIRESTORE USER DATA

ตรวจสอบ Structure เดิมก่อน

ถ้าจำเป็นต้องเพิ่ม User Profile Structure ให้ปรับเข้ากับของเดิม

Logical example:

```text
users/{userId}

email
displayName
role
status
createdAt
updatedAt
lastLoginAt
```

Possible status:

```text
ACTIVE
DISABLED
```

หากระบบเดิมมี Field ชื่ออื่นอยู่แล้ว:

**Reuse existing fields whenever practical**

ห้ามสร้าง Collection ซ้ำโดยไม่จำเป็น

---

# 18. EMAIL NORMALIZATION

Email ควร Normalize ก่อน Compare

เช่น:

```text
lowercase
trim whitespace
```

เพื่อป้องกัน:

```text
User@Company.com
```

กับ:

```text
user@company.com
```

ถูกมองเป็นคนละ Account โดยไม่จำเป็น

---

# 19. DUPLICATE USER PROTECTION

Admin ห้ามสร้าง Email ซ้ำ

หาก Email มีอยู่แล้ว:

ระบบต้องแจ้งว่า:

```text
User already exists.
```

และเสนอให้:

- Edit existing user
- Reactivate
- Change Role

แทนการสร้างอีก Account

---

# 20. DISABLE USER

เมื่อ Admin Disable User:

```text
status = DISABLED
```

User ต้องไม่สามารถเข้าระบบต่อได้

หาก User มี Session อยู่แล้ว ระบบต้องมีแนวทางทำให้ Permission ถูกปฏิเสธทันทีหรือเร็วที่สุดตาม Architecture ที่ใช้

ไม่ควร Delete Account เพื่อ Disable

---

# 21. USER DELETION

Phase นี้ **ไม่ต้องทำ Hard Delete User**

ใช้:

```text
DISABLED
```

เพื่อรักษา:

- Audit History
- Recipe History
- Changed By
- Created By

User เก่าอาจถูกอ้างอิงในข้อมูลระบบในอนาคต

---

# 22. LAST LOGIN

เมื่อ Login สำเร็จ:

Update:

```text
lastLoginAt
```

เพื่อให้ Admin ตรวจสอบการใช้งานได้

อย่า Update แบบที่ทำให้เกิด Write ที่ไม่จำเป็นจำนวนมากเกินไป

---

# 23. SESSION

ระบบต้องรองรับ:

- Persistent login ตามแนวทางที่เหมาะสม
- Logout
- Expired/invalid session handling

หลัง Logout:

User ต้องไม่สามารถกลับเข้า Protected Screen ผ่าน Browser Back แล้วใช้งานต่อได้

---

# 24. PROTECTED ROUTES

หน้าที่ต้อง Login ห้ามเปิดโดย Anonymous User

ตัวอย่าง:

```text
/dashboard
/recipes
/master-items
/users
/support
```

หากไม่ Login:

Redirect ไป Login

---

# 25. ROLE-BASED ROUTING

หลัง Login สามารถใช้ Role เพื่อกำหนด Navigation ที่เหมาะสม

### ADMIN

เห็น Administration Navigation

### TRAINER

ไม่เห็น User Administration / Price Management ที่ไม่มีสิทธิ์

### SALES

เห็นเฉพาะ Feature ที่ได้รับอนุญาต

แต่การซ่อน Navigation เป็น UX เท่านั้น

Security จริงต้องยังถูกตรวจที่ Backend / Firestore Rules

---

# 26. ERROR MESSAGES

ข้อความควรเข้าใจง่าย

ตัวอย่าง:

Incorrect Login:

```text
Email หรือรหัสผ่านไม่ถูกต้อง
```

Unauthorized Email:

```text
Email นี้ไม่ได้รับอนุญาตให้เข้าใช้งานระบบ
```

Disabled:

```text
บัญชีนี้ถูกระงับการใช้งาน กรุณาติดต่อผู้ดูแลระบบ
```

Password Reset:

```text
หาก Email นี้อยู่ในระบบ เราจะส่งคำแนะนำสำหรับตั้งรหัสผ่านใหม่ไปยัง Email
```

หลีกเลี่ยงการเปิดเผยข้อมูล Security ที่ไม่จำเป็น

---

# 27. ADMIN USER MANAGEMENT UI

สร้างหน้า User Management ที่เรียบง่าย

แสดง:

```text
Name
Email
Role
Status
Last Login
```

Actions:

```text
Add User
Change Role
Enable
Disable
Password Reset Action
```

Mobile-friendly

---

# 28. ADD USER UI

Admin กด:

```text
+ Add User
```

กรอก:

```text
Display Name
Email
Role
```

จากนั้น:

```text
Create / Approve User
```

ไม่ต้องมีช่อง:

```text
Password
Confirm Password
```

เพราะ User เป็นคนตั้ง Password เอง

---

# 29. NO SAMPLE DATA

ห้ามสร้าง Account เช่น:

```text
admin@example.com
sales@example.com
trainer@example.com
test@test.com
demo@demo.com
```

Production ต้องใช้ Existing Data หรือ User ที่ Admin สร้างจริงเท่านั้น

---

# 30. FIREBASE-SPECIFIC INSPECTION

ก่อน Implement ให้ตรวจอย่างน้อย:

```text
Firebase project configuration
Firestore database
Existing collections
Existing users
Current Firebase Authentication configuration
Firestore Security Rules
Firebase Hosting config if applicable
Cloud Functions if applicable
Environment variables
Existing Firebase indexes
```

อย่าเปลี่ยน Project ID หรือ Database โดยไม่จำเป็น

---

# 31. DO NOT MIGRATE DATABASE

Application ใช้ Google Firebase / Firestore

ห้ามย้ายไป:

- MySQL
- PostgreSQL
- Supabase
- MongoDB
- SQL Server
- Database Provider อื่น

ใน Phase นี้

Database schema ที่อธิบายใน Specification อื่นให้ถือเป็น Logical Model เท่านั้น

Implementation จริงต้องเข้ากับ Firestore Architecture เดิม

---

# 32. PHASE 1 TEST CASES

Claude ต้องทดสอบอย่างน้อย:

## Test 1

Approved Email + Correct Password

Expected:

```text
Login Success
```

---

## Test 2

Wrong Password

Expected:

```text
Login Rejected
```

---

## Test 3

Email ไม่ได้รับอนุญาต

Expected:

```text
Cannot create/access account
```

---

## Test 4

First-Time Approved User

Expected:

```text
User can securely set their own password
```

---

## Test 5

Forgot Password

Expected:

```text
Reset email is sent through the configured authentication flow
```

---

## Test 6

Disabled User

Expected:

```text
Access Rejected
```

---

## Test 7

SALES attempts Admin operation

Expected:

```text
Permission Denied
```

---

## Test 8

TRAINER attempts Price/Admin operation

Expected:

```text
Permission Denied
```

---

## Test 9

Admin adds duplicate Email

Expected:

```text
Duplicate prevented
```

---

## Test 10

Logout

Expected:

```text
Protected pages inaccessible
```

---

# 33. MOBILE TEST

ทดสอบ Login และ User Management อย่างน้อย:

```text
360px
390px
430px
Desktop
```

ต้องไม่มี:

- Button หลุดจอ
- Text ถูกตัด
- Form เกินหน้าจอ
- Horizontal Scroll ที่ไม่จำเป็น
- Modal ใช้งานไม่ได้

---

# 34. PHASE 1 ACCEPTANCE CRITERIA

Phase 1 ถือว่าผ่านเมื่อ:

- Existing Firebase Data ไม่เสียหาย
- Approved Email Model ทำงาน
- User ตั้ง Password เองได้อย่างปลอดภัย
- Email + Password Login ทำงาน
- Forgot Password ทำงาน
- Reset Password ผ่าน Email ทำงาน
- Public Registration ไม่มี
- Google Login ไม่มี
- Social Login ไม่มี
- ADMIN / TRAINER / SALES ถูกแยก
- Disabled User เข้าไม่ได้
- Permission ไม่พึ่งแค่ Frontend
- Existing User Data ยังอยู่ครบ
- Mobile Login ใช้งานได้

---

# 35. STOP AFTER PHASE 1

เมื่อทำ Phase 1 เสร็จ:

**ห้ามเริ่ม Phase 2 อัตโนมัติ**

ให้หยุดและส่ง Report ก่อน

Report ต้องประกอบด้วย:

### A. Existing System Found
สิ่งที่พบในระบบเดิม

### B. Changes Made
รายการที่แก้

### C. Firebase Changes
Collections / Rules / Auth Config / Functions ที่แตะ

### D. Data Migration
ถ้ามี ให้ระบุ

### E. Tests Performed
ผล Test แต่ละรายการ

### F. Existing Data Verification
ยืนยันว่าข้อมูลเดิมยังอยู่

### G. Issues / Risks
ปัญหาหรือข้อจำกัดที่พบ

### H. Screens / User Flow
สรุป Login / First-Time Setup / Forgot Password / Admin User Management ที่ทำเสร็จ

จากนั้นรอการตรวจรับก่อนเริ่ม Phase 2

---

# FINAL INSTRUCTION

Do not implement any later phase.

Complete only Phase 1: Authentication, approved-email access, first-time password setup, password reset, user management, and role-based security.

Preserve the existing Firebase / Firestore production data.

Do not create demo data.

Do not create Google Sign-In.

Do not create public registration.

Do not allow unauthorized email addresses to create accounts.

Users must set and manage their own passwords.

Admin controls which email addresses and roles are allowed to access the application.

After completing this phase, stop and provide the Phase 1 verification report before making any further application changes.