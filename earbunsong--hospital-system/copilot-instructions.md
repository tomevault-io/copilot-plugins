## hospital-system

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Spring Boot Hospital Management System (HMS)** built with modern Java practices, featuring role-based security, PostgreSQL database, and Thymeleaf templating. The system provides comprehensive hospital management capabilities including patient registration, doctor management, appointment scheduling, and visit documentation.

## Technology Stack

### Core Technologies
- **Java 17** - Modern LTS version with latest features
- **Spring Boot 3.5.6** - Core framework with auto-configuration
- **Maven 3.x** - Build and dependency management
- **PostgreSQL 42.7.7** - Primary production database
- **H2 Database** - In-memory database for testing and development
- **Thymeleaf 3.x** - Server-side templating engine
- **Spring Security 6.x** - Authentication and authorization framework

### Key Dependencies
- **Spring Data JPA** - Database abstraction with Hibernate
- **Spring Boot Starter Web** - RESTful web services
- **Spring Boot Starter Security** - Security configuration
- **Spring Boot Starter Validation** - Bean validation
- **Thymeleaf Extras SpringSecurity6** - Security integration
- **BCrypt** - Password encoding

## Build and Development Commands

### Prerequisites
- Java 17+ installed
- Maven 3.6+ or Maven wrapper included
- PostgreSQL database running on localhost:5432
- Database name: `hospital_management`

### Build Commands
```bash
# Build the project
./mvnw clean compile

# Run tests
./mvnw test

# Package the application
./mvnw package

# Run the application
./mvnw spring-boot:run

# Clean build and run
./mvnw clean spring-boot:run
```

### Development Workflow
1. **Start PostgreSQL database** on default port 5432
2. **Run the application** using Maven wrapper
3. **Access the application** at `http://localhost:8080`
4. **Login credentials**: admin/Admin@2024 (default admin user)

## Database Configuration

### PostgreSQL Setup
```properties
# application.properties
spring.datasource.url=jdbc:postgresql://localhost:5432/hospital_management
spring.datasource.username=postgres
spring.datasource.password=postgres
spring.datasource.driver-class-name=org.postgresql.Driver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

### Data Loading
The system automatically initializes with:
- **Default admin user**: `admin` with password `Admin@2024`
- **Role hierarchy**: ADMIN, STAFF, DOCTOR
- **Schema auto-creation**: Hibernate updates schema automatically

## Security Setup

### Authentication System
- **CustomUserDetailsService** implements Spring Security's UserDetailsService
- **BCrypt password encoding** for secure storage
- **Role-based access control** with hierarchical permissions

### Security Configuration
```java
// Key security features:
- Form-based login with custom login page
- Method-level security with @PreAuthorize
- Role-based URL access control
- CSRF protection with H2 console exception
- Session management with logout functionality
```

### User Roles and Permissions
| Role | Permissions | Access Level |
|------|------------|--------------|
| ADMIN | Full system access, user management | `/admin/**`, `/doctors/**`, `/users/**` |
| STAFF | Patient management, appointments | `/patients/**`, `/appointments/**` |
| DOCTOR | Schedule viewing, visit notes | `/appointments/my-schedule`, `/visit-notes/**` |

## Key Architectural Patterns

### 1. Layered Architecture
- **Controller Layer**: HTTP request handling with validation
- **Service Layer**: Business logic and transaction management
- **Repository Layer**: Data access with Spring Data JPA
- **Model Layer**: JPA entities with relationships

### 2. Security Implementation
- **Method-level security** using `@PreAuthorize` annotations
- **Role hierarchy** with Spring Security context
- **Custom authentication** with database-backed user details
- **Secure password storage** with BCrypt encoding

### 3. Data Relationships
```
User (1) ←→ (*) Role
User (1) ←→ (1) Doctor (Optional)
Patient (*) ←→ (*) Appointment ←→ (1) Doctor
Appointment (1) ←→ (1) VisitNote
```

## Thymeleaf Templating

### Template Structure
- **26 HTML templates** with Bootstrap 5 styling
- **Fragment-based layout** with reusable components
- **Spring Security integration** for role-based UI elements
- **Responsive design** with mobile-first approach

### Key Templates
- `fragments/layout.html` - Main navigation and structure
- `patients/list.html` - Patient management interface
- `doctors/list.html` - Doctor directory
- `appointments/list.html` - Appointment scheduling
- `login.html` - Authentication form

## Development Guidelines

### Code Standards
- **Constructor injection** preferred over field injection
- **Validation annotations** on all input entities
- **Spring Security best practices** implemented
- **Consistent naming conventions** throughout

### Testing
- **JUnit 5** for unit testing
- **Spring Security Test** for security testing
- **H2 in-memory database** for testing
- **Integration tests** with `@SpringBootTest`

### Logging
- **SLF4J** with parameterized logging
- **DEBUG level** for Spring Security (configured)
- **Formatted SQL** output for development

## Features and Functionality

### Core Features
1. **Patient Management**
   - CRUD operations for patient records
   - Search by name or phone number
   - Personal information storage

2. **Doctor Management**
   - Doctor profile management
   - Specialty assignment
   - Schedule tracking

3. **Appointment System**
   - Appointment scheduling
   - Status management (CONFIRMED, CANCELLED, COMPLETED)
   - Doctor-patient relationship management

4. **Visit Notes**
   - Medical documentation
   - Appointment-linked notes
   - Doctor-specific access

5. **User Management**
   - Role-based user accounts
   - Secure authentication
   - Admin user management

## Database Schema

### Core Tables
- `users` - User accounts with Spring Security integration
- `roles` - System roles (ADMIN, STAFF, DOCTOR)
- `user_roles` - Many-to-many user-role mapping
- `patients` - Patient information
- `doctors` - Doctor profiles linked to users
- `appointments` - Appointment scheduling
- `visit_notes` - Medical documentation

## Development Environment Setup

### Local Development
1. **Install Java 17**
2. **Set up PostgreSQL** database
3. **Create database**: `hospital_management`
4. **Run application**: `./mvnw spring-boot:run`
5. **Access at**: `http://localhost:8080`

### Default Login
- **Username**: `admin`
- **Password**: `Admin@2024`
- **Role**: ADMIN (full system access)

---
> Source: [Earbunsong/hospital_system](https://github.com/Earbunsong/hospital_system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
