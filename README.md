<p align="center">
  <img src="StudentCouncil.Frontend/public/favicon.svg" width="100" alt="Логотип" />
</p>

<h1 align="center">Платформа студенческого совета СГН</h1>

<p align="center">
  Веб-приложение для управления студенческим советом: участники, мероприятия, бейджи, аналитика.
</p>

<p align="center">
  <a href="https://studsovetsgn.ru"><img src="https://img.shields.io/badge/studsovetsgn.ru-0CBFA1" /></a>
  <img src="https://img.shields.io/badge/.NET-8.0-512BD4" />
  <img src="https://img.shields.io/badge/React-19-61DAFB" />
  <img src="https://img.shields.io/badge/PostgreSQL-16-4169E1" />
  <img src="https://img.shields.io/badge/Docker-ready-2496ED" />
</p>

---

## О проекте

Внутренняя платформа для студенческого совета СГН МГТУ им. Н.Э. Баумана. Заменяет ручной учёт в таблицах и мессенджерах единой системой с ролевой моделью, геймификацией и аналитикой.

**Что умеет:**

- Учёт участников с ролями (Admin / Leader / Member)
- Организация мероприятий: регистрация, явка, бюджет
- Выдача бейджей с PDF-подтверждением
- Аналитика: статистика, топы, динамика по месяцам
- Двухфакторная аутентификация (TOTP)

---

## Структура

```text
StudentCouncil/
├── StudentCouncil.Data/      # DbContext, модели, миграции
├── StudentCouncil.Logic/     # DTO, интерфейсы, сервисы
├── StudentCouncil.WebApi/    # Контроллеры, Program.cs, wwwroot
├── StudentCouncil.Frontend/  # React SPA (Vite)
├── docker-compose.yml
├── Dockerfile
└── openapi.yaml
```

##  Документация API

- [Swagger UI](https://digital-sgn.github.io/StudentCouncilPlatform-API)
- [openapi.yaml](openapi.yaml) — спецификация в репозитории
- Swagger в dev-режиме: `http://localhost:8080/swagger`

## Стек

**Backend**  
![C#](https://img.shields.io/badge/C%23-239120?logo=c-sharp&logoColor=white)
![ASP.NET Core](https://img.shields.io/badge/ASP.NET_Core_8-512BD4?logo=.net&logoColor=white)
![EF Core](https://img.shields.io/badge/EF_Core-512BD4)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?logo=postgresql&logoColor=white)

**Frontend**  
![React](https://img.shields.io/badge/React_19-61DAFB?logo=react&logoColor=black)
![Vite](https://img.shields.io/badge/Vite-646CFF?logo=vite&logoColor=white)
![Bootstrap](https://img.shields.io/badge/Bootstrap_5-7952B3?logo=bootstrap&logoColor=white)

**Инфраструктура**  
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?logo=nginx&logoColor=white)
![Ubuntu](https://img.shields.io/badge/Ubuntu_22.04-E95420?logo=ubuntu&logoColor=white)

---

## Архитектура

### Схема БД
```mermaid
erDiagram
    AspNetUsers ||--o{ Events : "ResponsibleUser"
    AspNetUsers ||--o{ Badges : "User"
    Events ||--o{ Badges : "Event"
    AspNetUsers ||--o{ AspNetUserRoles : ""
    AspNetRoles ||--o{ AspNetUserRoles : ""

    AspNetUsers {
        int Id PK
        string UserName
        string NormalizedUserName
        string Email
        string NormalizedEmail
        bool EmailConfirmed
        string PasswordHash
        string SecurityStamp
        string ConcurrencyStamp
        string PhoneNumber
        bool PhoneNumberConfirmed
        bool TwoFactorEnabled
        datetime LockoutEnd
        bool LockoutEnabled
        int AccessFailedCount
        string FirstName "max 20, required"
        string LastName "max 20, required"
        string Patronymic "max 20"
        string Group "max 10"
        datetime BirthDate
        string Telegram "max 20"
        string ClothingSize "max 10"
        string AvatarPath
        int Balance
        int TotalPointsEarned
        int ExperiencePoints
        int Level
        int EventsAttended
        int EventsOrganized
        int TasksCompleted
        bool IsActive
        datetime JoinedAt
        datetime LastActivityDate
    }

    Events {
        int Id PK
        string Title "max 100, required"
        string Description "max 400"
        datetime EventDate "required"
        decimal Budget
        string Location "max 100, required"
        int RegisteredParticipants
        int ActualParticipants
        string RegistrationLink "max 200"
        string PhotoPath "max 200"
        int Status "Upcoming|Completed|Cancelled"
        int ResponsibleUserId FK "required"
        datetime CreatedAt
        bool IsDeleted
    }

    Badges {
        int Id PK
        int UserId FK "required"
        int EventId FK "required"
        string Role "max 100, required"
        string FilePath "max 500, required"
        datetime CreatedAt
    }

    AspNetRoles {
        int Id PK
        string Name
        string NormalizedName
        string ConcurrencyStamp
    }

    AspNetUserRoles {
        int UserId FK
        int RoleId FK
    }

    AspNetUserClaims {
        int Id PK
        int UserId FK
        string ClaimType
        string ClaimValue
    }

    AspNetRoleClaims {
        int Id PK
        int RoleId FK
        string ClaimType
        string ClaimValue
    }

    AspNetUserLogins {
        string LoginProvider PK
        string ProviderKey PK
        int UserId FK
    }

    AspNetUserTokens {
        int UserId PK
        string LoginProvider PK
        string Name PK
        string Value
    }
```


### Архитектура бэкенда

```mermaid
flowchart TB
    subgraph Web["StudentCouncil.Web (ASP.NET Core 8)"]
        BC[BaseController<br/>HandleServiceResult]
        AC[AccountController<br/>/api/account]
        UC[UserController<br/>/api/users]
        EC[EventController<br/>/api/events]
        BDC[BadgeController<br/>/api/badges]
    end

    subgraph Logic["StudentCouncil.Logic"]
        SR[ServiceResult / ServiceResult&lt;T&gt;]
        MAP[Mapper]
        subgraph Interfaces
            IUS[IUserService]
            IES[IEventService]
            IBS[IBadgeService]
            IFS[IFileStorageService]
            ILS[ILoggerService]
        end
        subgraph Services
            US[UserService]
            ES[EventService]
            BS[BadgeService]
            FS[FileStorageService]
            LS[FileLoggerService]
        end
    end

    subgraph Data["StudentCouncil.Data"]
        DB[AppDbContext<br/>IdentityDbContext]
        subgraph Models
            UM[User]
            EM[Event]
            BM[Badge]
        end
    end

    PG[(PostgreSQL)]

    AC --> IUS
    UC --> IUS
    EC --> IES
    BDC --> IBS
    BDC --> IFS

    IUS -.реализует.-> US
    IES -.реализует.-> ES
    IBS -.реализует.-> BS
    IFS -.реализует.-> FS
    ILS -.реализует.-> LS

    US --> MAP
    ES --> MAP
    BS --> MAP
    US --> DB
    ES --> DB
    BS --> DB
    DB --> UM
    DB --> EM
    DB --> BM
    DB --> PG

    style Web fill:#512BD4,color:#fff
    style Logic fill:#0CBFA1,color:#04120e
    style Data fill:#148C9C,color:#fff
```


### Зависимости сервисов

```mermaid
flowchart LR
    subgraph UserService
        US[UserService]
        UM[UserManager&lt;User&gt;]
        UFS[IFileStorageService]
        ULS[ILoggerService]
    end

    subgraph EventService
        ES[EventService]
        ECtx[AppDbContext]
        EFS[IFileStorageService]
        ELS[ILoggerService]
    end

    subgraph BadgeService
        BS[BadgeService]
        BCtx[AppDbContext]
        BFS[IFileStorageService]
        BLS[ILoggerService]
    end

    US --> UM
    US --> UFS
    US --> ULS

    ES --> ECtx
    ES --> EFS
    ES --> ELS

    BS --> BCtx
    BS --> BFS
    BS --> BLS
```

### DI-регистрация

```mermaid
flowchart LR
    DI[ServiceCollection]
    DI -->|Transient| IUS[IUserService → UserService]
    DI -->|Transient| IES[IEventService → EventService]
    DI -->|Transient| IBS[IBadgeService → BadgeService]
    DI -->|Transient| IFS[IFileStorageService → FileStorageService]
    DI -->|Singleton| ILS[ILoggerService → FileLoggerService]
    DI -->|Scoped| DB[AppDbContext]
    DI -->|Identity| IM[UserManager, SignInManager, RoleManager]
```

### ServiceResult — паттерн ответа

```mermaid
flowchart TB
    Service[Сервис] -->|Ok / Created| Success
    Service -->|BadRequest / NotFound| ClientError
    Service -->|Forbidden / Unauthorized| AuthError
    Service -->|Conflict / InternalError| Other

    Success -->|200 / 201| JSON["{ data } или { message }"]
    ClientError -->|400 / 404| JSONErr["{ error: message }"]
    AuthError -->|401 / 403| JSONErr
    Other -->|409 / 500| JSONErr
```

### Аутентификация

```mermaid
flowchart TB
    Login[POST /api/account/login] --> Find[FindByEmailAsync]
    Find --> CheckPwd[CheckPasswordAsync]
    CheckPwd --> TwoFA{2FA включена?}
    TwoFA -->|Нет| Setup[Вернуть 402<br/>+ TOTP-ключ]
    TwoFA -->|Да| SignIn[PasswordSignInAsync]
    SignIn -->|Succeeded| OK[200 + cookie]
    SignIn -->|RequiresTwoFactor| Code[403<br/>+ requiresTwoFactorCode]
    SignIn -->|Fail| Bad[401]

    Code --> Verify[POST /2fa/verification]
    Verify --> TOTP[TwoFactorAuthenticatorSignInAsync]
    TOTP -->|OK| OK2[200 + cookie]

    style Setup fill:#ffd700,color:#04120e
    style Code fill:#ffd700,color:#04120e
```

---

## Локальный запуск

---

## Docker

---


## Ссылки

- Сайт: [studsovetsgn.ru](https://studsovetsgn.ru)
- Справочник отдела: [Handbook](https://github.com/Digital-SGN/Handbook)
- Организация на GitHub: [Digital-SGN](https://github.com/Digital-SGN)

---

<p align="center">
  <sub>Отдел цифрового развития ССФ СГН · 2026</sub>
</p>
