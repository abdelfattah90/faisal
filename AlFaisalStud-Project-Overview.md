# مربط الفيصل (Al-Faisal Stud) — نظام الحجز وإدارة الاشتراكات

## 1. نظرة عامة على المشروع

**الاسم التجاري:** مربط الفيصل اف اند ان (ALFAISAL STUD F AND N FOR ARABIA HORSES)

**طبيعة النظام:** نظام **Booking + Subscription Management** لخدمات مربط خيل، وليس نظام "بيع وشراء" تقليدي (E-commerce بالمعنى الكلاسيكي). الفرق الجوهري: المنتجات هنا ليست سلعًا مادية، بل **باقات حصص واشتراكات لها صلاحية زمنية ورصيد قابل للاستهلاك**.

---

## 2. تحليل النطاق التجاري (Domain Analysis)

الخدمات مقسّمة إلى أربع فئات رئيسية:

### أ. باقات تدريب فردية (Individual Subscriptions)
- قفز حواجز، ركوب خيل أرضي، رماية أرضية
- كل باقة: عدد حصص ثابت + مدة صلاحية محددة (أسبوعين إلى شهرين ونصف)

### ب. باقات جماعية (Group Packages)
- باقات لـ 2/3/4 أشخاص، خاصة بالركوب الأرضي فقط، صلاحية شهر واحد

### ج. اشتراكات الرحلات (Trip Subscriptions)
- شهري / 6 أشهر / سنوي، بمعدل ثابت شهريًا (8 رحلات/شهر)
- **قاعدة مهمة:** لا يحق ترحيل الرحلات غير المستخدمة للشهر التالي (No Rollover Rule)

### د. خدمات فردية (À la carte / Single Services)
- تذاكر منفردة بدون اشتراك: حصة ركوب، قفز حواجز، رماية، سباحة، ركوب خيل قزم للأطفال

### قيود تشغيلية إضافية
- **الدفع بالتقسيط:** دعم Tabby (Buy Now Pay Later) على معظم الباقات
- **أوقات نسائية حصرية:** الأحد والثلاثاء 5:30م–10:30م، المربط يكون للسيدات فقط، ويُمنع دخول المرافقين الرجال — قيد على مستوى الـ Booking/Scheduling logic

---

## 3. المستخدمون والصلاحيات (Roles & Permissions)

النظام يحتوي على ثلاثة أنواع مستخدمين رئيسية (نمط RBAC):

| Role | الوصف | الصلاحيات الأساسية |
|---|---|---|
| **SuperAdmin** | صاحب النظام | تحكم كامل: إدارة الـ Admins/SubAdmins، الإعدادات العامة، التقارير المالية، كل CRUD operations |
| **Admin / SubAdmin** | فريق الإدارة | إدارة الباقات، الحجوزات، الاشتراكات، المستخدمين (SubAdmin بصلاحيات أضيق) |
| **Coach** | المدرب | تحديد أوقاته المتاحة للتدريب، متابعة الـ Trainees المخصصين له (حضور/تقدم) — **يعمل من داخل Admin Dashboard بنفس الـ Route، بواجهة/صلاحيات مختلفة** |
| **Trainee (User)** | مستخدم الموقع | تصفح الباقات، حجز حصص، إدارة اشتراكه ورصيده، الدفع |

**ملاحظة معمارية:** الـ Coach ليس تطبيقًا منفصلاً، بل Role إضافي داخل Admin Dashboard مع navigation/permissions مختلفة عن Admin. تفاصيل الـ permissions الدقيقة (Policy-based Authorization) ستُحسم لاحقًا.

**الـ Routing المعتمد:** بدون subdomain — نفس الدومين بمسار داخلي: `site.com/auth/admin/`

---

## 4. التكنولوجيا المعتمدة (Tech Stack)

| الطبقة | التقنية | ملاحظات |
|---|---|---|
| **Backend** | ASP.NET Core Web API | API فقط، منفصل عن الـ Frontend |
| **Database** | SQL Server | مناسب لطبيعة العلاقات المعقدة (اشتراكات، حجوزات، أرصدة) |
| **ORM** | Entity Framework Core | |
| **Frontend (Store)** | React + TypeScript | مشروع مستقل لموقع العملاء (Trainees) |
| **Frontend (Admin)** | React + TypeScript | مشروع منفصل تمامًا، يخدم Admin/SubAdmin/Coach بواجهات مختلفة حسب الـ Role |
| **Auth** | JWT (Access + Refresh Tokens) | Role-based: SuperAdmin, Admin, SubAdmin, Coach, Trainee |
| **Payment** | Tabby (تقسيط) | + بوابة دفع كاش/بطاقة عادية (لم تُحدد بعد) |

---

## 5. Architecture Pattern: Modular Monolith

**القرار:** Modular Monolith بتقسيم Vertical Slice (بحسب الـ Domain)، وليس N-Layer التقليدي (تقسيم بحسب نوع الملف).

**السبب:** المشروع يحتوي على domains واضحة الحدود (Booking, Subscription, Plans, Payment)، وكل واحد له قواعده الخاصة. Vertical Slice يسمح بتطوير واختبار كل موديول بمعزل عن الآخر، وهذا مهم مع نمو المشروع.

### الهيكل المخطط (Solution Structure)
```
AlFaisalStud/
├── backend/
│   ├── AlFaisalStud.sln
│   ├── src/
│   │   ├── AlFaisalStud.API/                  (entry point, Program.cs)
│   │   ├── AlFaisalStud.Modules.Auth/
│   │   ├── AlFaisalStud.Modules.Plans/
│   │   ├── AlFaisalStud.Modules.Booking/
│   │   ├── AlFaisalStud.Modules.Subscription/
│   │   ├── AlFaisalStud.Modules.Payment/
│   │   └── AlFaisalStud.Shared/               (DTOs/interfaces مشتركة)
│   └── tests/
├── frontend-store/       (React + TS — موقع العملاء)
└── frontend-admin/       (React + TS — لوحة التحكم، منفصلة بالكامل)
```

كل موديول = مشروع C# مستقل (Class Library) يُضاف كـ project reference للـ API، لعزل حقيقي على مستوى الـ compiler لا مجرد تنظيم مجلدات.

---

## 6. الموديولات المخطط لها

1. **Auth Module** — تسجيل/دخول، JWT، الأدوار (SuperAdmin, Admin, SubAdmin, Coach, Trainee)
2. **Plans Module** — إدارة الباقات والخدمات (الأسعار، المدد، عدد الحصص)
3. **Booking Module** — حجز الحصص/الرحلات، تحديد Coach لأوقاته المتاحة، منطق الأوقات النسائية
4. **Subscription Module** — رصيد المستخدم، صلاحية الاشتراك، قاعدة عدم الترحيل
5. **Payment Module** — تكامل Tabby + بوابة دفع عادية

**ملاحظة:** متابعة الـ Coach لتقدم/حضور الـ Trainees تُبنى فوق Booking + Subscription Modules، وليست موديولاً منفصلاً في هذه المرحلة.

---

## 7. خارطة الطريق (Roadmap) — بناء تدريجي

```
Phase 0: Project Skeleton (Solution + Folders)
Phase 1: Database Design (Core Entities)          ← الخطوة التالية
Phase 2: Backend - Auth + Plans Module
Phase 3: Frontend Store - Setup + Auth Flow
Phase 4: Backend - Booking + Subscription Modules
Phase 5: Frontend Store - Booking Flow
Phase 6: Admin Dashboard
Phase 7: Payment Integration (Tabby)
Phase 8: Deployment
```

**ملاحظة على الترتيب:** تصميم الـ Database Entities يسبق هيكلة المجلدات النهائية، لأن العلاقات بين الكيانات هي ما يحدد شكل الموديولات بدقة.

---

## 8. القرارات المفتوحة

- تصميم الـ Database Entities بالتفصيل (Core Entities: User, Role, Package, Subscription, Booking)
- بوابة دفع إضافية بجانب Tabby (Stripe / PayPal / Paymob / Fawry)
- File storage للصور (Local / Azure Blob / AWS S3 / Cloudinary)
- نطاق كامل ودقيق لصفحات Admin Dashboard (Admin / SubAdmin / Coach)
- تفاصيل الـ Policy-based Permissions الدقيقة (لاحقًا، ليست الآن)
