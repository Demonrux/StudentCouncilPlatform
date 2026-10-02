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


### Архитектура слоёв

```mermaid
flowchart TD
    Client[Браузер<br/>React SPA] -->|HTTPS| Nginx[Nginx<br/>reverse proxy + статика]
    Nginx -->|/api /avatars /badges| WebApi[StudentCouncil.WebApi<br/>контроллеры]
    Nginx -->|/| Static[Статика React<br/>/var/www/studsovetsgn]
    
    WebApi --> Logic[StudentCouncil.Logic<br/>сервисы, DTO]
    Logic --> Data[StudentCouncil.Data<br/>EF Core, модели]
    Data --> Postgres[(PostgreSQL)]
    
    WebApi -.->|файлы| Volume[Docker volume<br/>backend_wwwroot]
    
    style Client fill:#0CBFA1,color:#04120e
    style Nginx fill:#148C9C,color:#fff
    style WebApi fill:#512BD4,color:#fff
    style Logic fill:#512BD4,color:#fff
    style Data fill:#512BD4,color:#fff
    style Postgres fill:#4169E1,color:#fff
```


### Компоненты бэкенда 

```mermaid
flowchart LR
    subgraph Controllers
        AC[AccountController]
        UC[UsersController]
        EC[EventsController]
        BC[BadgesController]
    end

    subgraph Services
        US[UserService]
        ES[EventService]
        BS[BadgeService]
        FS[FileStorageService]
    end

    subgraph Interfaces
        IUS[IUserService]
        IES[IEventService]
        IBS[IBadgeService]
        IFS[IFileStorageService]
    end

    AC --> IUS
    UC --> IUS
    EC --> IES
    BC --> IBS
    BC --> IFS

    IUS -.-> US
    IES -.-> ES
    IBS -.-> BS
    IFS -.-> FS

    style AC fill:#512BD4,color:#fff
    style UC fill:#512BD4,color:#fff
    style EC fill:#512BD4,color:#fff
    style BC fill:#512BD4,color:#fff
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
