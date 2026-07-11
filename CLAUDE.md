# CLAUDE.md

## Project Overview

Headwind MDM is an open-source Mobile Device Management (MDM) platform for Android devices. It is a multi-module Maven project written in Java 8, deployed as a WAR on Apache Tomcat with PostgreSQL as the database.

## Build & Run Commands

### Build the project
```bash
# Build all modules
mvn clean package

# Build skipping tests
mvn clean package -DskipTests

# Build with specific properties
cp server/build.properties.example server/build.properties
# Edit server/build.properties with your settings
mvn validate      # Apply property filtering to context.xml
mvn clean package
```

### Run single test
```bash
mvn test -pl common -Dtest=YourTestClass
```

### Frontend build (handled automatically by frontend-maven-plugin)
```bash
# Frontend is built as part of Maven package phase
# Manual frontend build (in server/webtarget/):
npm install
grunt build
```

## Architecture

### Maven Module Structure

| Module | Packaging | Purpose |
|--------|-----------|---------|
| `common` | jar | Core shared library — domain models, DAOs, MyBatis mappers, REST filters, events |
| `jwt` | jar | JWT authentication (jjwt 0.9.1) |
| `notification` | jar | Push notification system (ActiveMQ MQTT + HTTP Long Polling) |
| `server` | war | Main web application (final name: `launcher.war`), includes AngularJS frontend |
| `plugins/` | pom | Parent POM for all plugin sub-modules |
| `swagger/ui` | war | Swagger API documentation UI |

### Plugin Modules (under `plugins/`)
- `platform` — Plugin infrastructure (classpath scanning, registry)
- `devicelog/core` + `devicelog/postgres` — Device log collection & PostgreSQL persistence
- `audit` — API request/response audit logging
- `deviceinfo` — Device dynamic info (GPS, WiFi, mobile data, battery)
- `messaging` — Messaging between server and devices
- `push` — Scheduled push notifications
- `xtra` — Miscellaneous features

### Dependency Injection

Uses **Google Guice 4.2.2** with HK2-Guice bridge for Jersey integration.

Key Guice modules loaded in `Initializer.java`:
1. `PersistenceModule` — MyBatis SqlSessionFactory, maps `com.hmdm.persistence.mapper` package
2. `LiquibaseModule` — Database schema migrations
3. `ConfigureModule` — Binds Tomcat `context-param` values as Guice `@Named` constants
4. `MainRestModule` — CORS filter for `/rest/*` and `/api/*`
5. `PublicRestModule` — HSTS filter, public IP filter, public resource bindings
6. `PrivateRestModule` — JWT filter, auth filter, private IP filter, private resource bindings
7. Notification modules (persistence, liquibase, REST, MQTT config)
8. Plugin modules (persistence, liquibase, REST)

### Application Startup Flow

1. Tomcat loads `web.xml` → `GuiceServletContextListener` (`Initializer.java`)
2. Initializer creates Guice Injector with all modules
3. Jersey scans `com.hmdm` for `@Path`-annotated resources
4. Liquibase runs database migrations
5. Background tasks start (notification engine, MQTT, plugin initialization)

### Key Source Packages

```
com.hmdm/
├── persistence/
│   ├── domain/        # Entity classes (Device, Application, Configuration, User, etc.)
│   ├── mapper/        # MyBatis mapper interfaces
│   └── dao/           # DAO implementations
├── rest/
│   ├── resource/      # JAX-RS REST endpoint classes
│   ├── filter/        # Servlet filters (Auth, CORS, HSTS, IP filtering)
│   └── json/          # Request/response DTOs and view models
├── guice/
│   └── module/        # Guice configuration modules
├── auth/              # Authentication interfaces
├── event/             # Event system (EventService, EventListener)
├── task/              # Background tasks
└── security/
    └── jwt/           # JWT filter implementation
```

### REST API Architecture

- **Public endpoints** (`/rest/public/*`): No authentication required — login, device sync, QR code, public downloads
- **Private endpoints** (`/rest/private/*`): Require JWT + session auth + IP filter
- **Plugin endpoints** (`/rest/plugin/*`): Plugin-specific endpoints
- **File downloads** (`/files/*`): Direct file serving via `DownloadFilesServlet`

### Push Notification System

Two modes, selected by Guice configuration:
1. **MQTT** (primary): Embedded ActiveMQ broker on port 31000, with `PushSenderMqtt` and rate-limiting via `MqttThrottledSender`
2. **Long Polling** (fallback): `LongPollingServlet` + `PushSenderPolling`

### Plugin System

Plugins implement `PluginConfiguration` interface and register:
- Persistence modules (MyBatis mappers)
- Liquibase modules (DB migrations)
- REST modules (JAX-RS resources)
- Task modules (background tasks)

Discovery via ClassGraph classpath scanning at startup.

## Configuration

All configuration is via Tomcat `<context-param>` in `context.xml`:

**Template**: `server/conf/context.xml` (Maven resource filtering with `${variable}` placeholders)
**Properties**: `server/build.properties` (created from `server/build.properties.example`)

Key params: `jdbc.*`, `base.directory`, `files.directory`, `base.url`, `usage.scenario`, `mqtt.*`, `smtp.*`, `jwt.*`, `ldap.*`, `hash.secret`, `aapt.command`, `rebranding.*`

## Database

- **Database**: PostgreSQL
- **ORM**: MyBatis 3.5.3 with XML mappers
- **Migrations**: Liquibase (`server/src/main/resources/liquibase/db.changelog.xml`)
- **Init SQL**: `install/sql/hmdm_init.en.sql` (English), `hmdm_init.ru.sql` (Russian)

## Frontend

- **Framework**: AngularJS 1.x (module: `headwind-kiosk`)
- **Entry**: `server/src/main/webapp/app/app.js`
- **Dependencies**: ngResource, ngCookies, ui.bootstrap, ui.router, chart.js, ngIdle
- **i18n**: 13 locales (en_US, ru_RU, fr_FR, pt_PT, ar_AE, es_ES, de_DE, zh_TW, zh_CN, ja_JP, tr_TR, vi_VN, it_IT)
- **Build**: Grunt + Bower via `frontend-maven-plugin`

## Important Notes

- Java 8 source/target throughout — no Java 9+ features
- WAR final name is `launcher.war`
- Server depends on Tomcat for deployment; context parameters are set in Tomcat's `context.xml`
- The `usage.scenario` parameter controls multi-tenant (`shared`) vs single-tenant (`private`) mode
- MQTT uses embedded ActiveMQ; external MQTT broker can be configured via `mqtt.external` parameter
- Plugins are loaded at runtime from classpath — no hot-deploy, requires restart
