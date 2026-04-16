## sixsense

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sixsense is a Korean social community platform built with two modules:
- **Spring/** — Primary application: Spring MVC 5.0.7 + MyBatis 3.3.1 + MySQL, packaged as a WAR
- **JSP/** — Legacy e-commerce module using a raw JSP Front Controller pattern

## Build & Run Commands

```bash
# Build (from Spring directory)
cd Spring && mvn clean package

# Run tests
cd Spring && mvn test

# Run a single test class
cd Spring && mvn -Dtest=ClassName test

# Deploy: the WAR is output to Spring/target/. Deploy to Tomcat or similar servlet container.
```

## Architecture (Spring Module)

**Request flow:** JSP views → Controllers (`*.do` URL pattern) → Services → DAOs → MyBatis → MySQL (`leedb`)

### Layer structure under `Spring/src/main/java/com/spring/`:

- **controller/** — HTTP request handlers (40+ controllers). URL pattern: `*.do`
- **biz/{domain}/** — Each domain follows the pattern: `VO` + `Service` interface + `ServiceImpl` + `DAO`
  - **board** — Community posts (main table: `BOARD2`)
  - **member** — Users, email auth (`MailSendService`, `MailUtils`)
  - **comment** — Post comments
  - **heart** — Like/heart system
  - **report** — Content reporting/moderation
  - **tag** — Post tagging
  - **chat** — WebSocket real-time chat (`ChatServer`)
  - **common** — AES256 encryption, AOP advice classes

### Key configuration files:

| File | Purpose |
|------|---------|
| `Spring/src/main/resources/applicationContext.xml` | DataSource, MyBatis, mail, AOP, transaction manager |
| `Spring/src/main/webapp/WEB-INF/DispatcherServlet-servlet.xml` | Component scan, i18n, locale, exception mapping |
| `Spring/src/main/webapp/WEB-INF/web.xml` | Servlet mapping (`*.do`), encoding filter, XSS filter |
| `Spring/src/main/resources/mappings/board-mapping.xml` | MyBatis SQL mappings |
| `Spring/src/main/resources/sql-map-config.xml` | MyBatis type aliases and mapper config |

### Cross-cutting concerns:
- **XSS protection:** Lucy XSS Servlet Filter (Naver) applied on all routes
- **i18n:** Korean/English via `messageSource_ko.properties` / `messageSource_en.properties`
- **AOP:** Before/After/AfterReturning/AfterThrowing advice defined in `biz/common/`
- **Encryption:** AES256 via `AES256Util` in `biz/common/`

## Conventions

- Java 11 target (property), though maven-compiler-plugin is set to source/target 1.6
- Views are JSP files in `Spring/src/main/webapp/` (not under WEB-INF)
- Controllers map to `*.do` URLs
- Service layer always uses interface + Impl pattern
- MyBatis mappings live in `src/main/resources/mappings/`
- Database credentials and mail config are in `applicationContext.xml`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cochon-msss) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
