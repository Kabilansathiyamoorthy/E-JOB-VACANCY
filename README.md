# E-Job Vacancy

A complete, production-style **job vacancy and recruitment platform** covering every professional domain — built with a **Flutter (Android-first)** mobile app, a **Java Spring Boot** REST backend, and a **MySQL** database.

Three roles, one platform:

| Role | What they do |
|---|---|
| **Job Seeker** | Profile, resume, job search with filters, apply, save, track applications, recommendations, notifications, report jobs |
| **Recruiter / Company** | Company profile + logo, post/edit/delete jobs (admin-approved), review applicants, skill matching, shortlist → interview → select |
| **Admin** | Platform stats, company verification, job approval, user/block management, domains & skills catalog, report review, analytics |

## Technology Stack

- **Mobile:** Flutter, Dart, Material 3, provider state management, http REST client, flutter_secure_storage (JWT), image_picker + file_picker (logo/avatar/resume)
- **Backend:** Java 17, Spring Boot 3.5, Spring Security (JWT + roles), Spring Data JPA / Hibernate, Bean Validation, BCrypt, Maven
- **Database:** MySQL 8 (InnoDB, utf8mb4). H2 in-memory (MySQL mode) for tests.

## Architecture

```text
Flutter Mobile App
        ↓  REST + JWT
   Spring Boot Controllers
        ↓
      Services            ← business rules: matching, recommendations,
        ↓                    completion, notifications, ownership checks
    Repositories (JPA)
        ↓
      MySQL
```

```text
EJobVacancy/
├── mobile/          Flutter app (lib/core, models, services, repositories, providers, screens, widgets)
├── backend/         Spring Boot API (controller, service, repository, entity, dto, security, config, scheduler, exception)
├── database/        schema.sql — reference DDL + seed catalog
├── docs/            README.md (deep dive), API.md, DATABASE.md
└── .env.example     environment template (secrets never committed)
```

## Quick Start

### 1. Database

```bash
mysql -u root -p < database/schema.sql    # optional: app auto-creates schema via Hibernate
```

### 2. Backend

```bash
cp .env.example .env                       # then edit EJOB_DB_PASSWORD etc.
cd backend
mvn spring-boot:run                        # or: java -jar target/ejob-backend-1.0.0.jar
```

API is served at `http://localhost:8080`. Sample data (below) is seeded on first boot.

### 3. Mobile

```bash
cd mobile
flutter pub get
flutter run --dart-define=EJOB_API_BASE_URL=http://10.0.2.2:8080
# 10.0.2.2 = host loopback from the Android emulator; use your LAN IP for a physical device
```

### Build APK

```bash
cd mobile
flutter build apk --release --dart-define=EJOB_API_BASE_URL=https://your-api-host
# output: build/app/outputs/flutter-apk/app-release.apk
```

## Test Accounts (seeded sample data)

| Role | Email | Password | Notes |
|---|---|---|---|
| Admin | `admin@ejob.com` | `admin123` | Full admin dashboard |
| Recruiter | `recruiter@abc.com` | `recruiter123` | ABC Technologies — verified |
| Recruiter | `recruiter@xyz.com` | `recruiter123` | XYZ Solutions — verified |
| Recruiter | `recruiter@nova.com` | `recruiter123` | Nova Manufacturing — **pending verification** |
| Seeker | `seeker@ejob.com` | `seeker123` | Kabilan S — Java profile, has application |
| Seeker | `arun@ejob.com` | `seeker123` | Frontend profile |
| Seeker | `divya@ejob.com` | `seeker123` | UI/UX profile |

Seeded content: 33 domains, ~36 skills, 4 jobs (one pending approval, one from the unverified company), 3 applications with status history, notifications. All identities are fictional.

## Demo Workflow (end-to-end)

1. **Recruiter** registers → completes company profile (awaits verification).
2. **Admin** verifies the company (`Admin → Verify Companies`).
3. **Recruiter** posts a job (`Post Job` → "submitted for admin approval").
4. **Admin** approves the job (`Job Moderation → Approve`) — it becomes publicly searchable.
5. **Seeker** registers, completes profile (skills, education, resume PDF), searches & filters jobs.
6. **Seeker** applies (cover letter + resume) — duplicate/deadline-closed applications are blocked.
7. **Recruiter** opens `Applicants`, sees transparent skill match, shortlists / schedules interview / selects.
8. **Seeker** receives notifications and tracks the pipeline in `Applications`; can withdraw before selection.
9. **Seeker** can report suspicious jobs; **Admin** resolves or dismisses reports and can remove fake jobs.
10. Jobs automatically expire after their deadline (hourly scheduler) and recruiters get expiry warnings.

## Security Highlights

- JWT (HMAC, ≥32-byte base64 secret from env), stateless API, role-based URL rules + method checks
- BCrypt password hashing; blocked users instantly rejected by the auth filter
- Ownership validation: recruiters only touch their own jobs/applicants; seekers only their own applications
- Private file storage: resumes/avatars served only through authorized endpoints (owner, authorized recruiters, admin)
- Upload validation: PDF magic-byte check, image whitelist, 10 MB cap, randomized file names

## Documentation

- [docs/README.md](docs/README.md) — features, setup, environment, deployment deep dive
- [docs/API.md](docs/API.md) — full REST API reference
- [docs/DATABASE.md](docs/DATABASE.md) — schema, ER description, indexes
