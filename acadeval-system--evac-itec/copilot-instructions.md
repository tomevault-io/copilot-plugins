## evac-itec

> Este archivo es un resumen técnico generado por Gemini para una referencia rápida sobre la estructura, tecnologías y comandos del proyecto.

# Hoja de Referencia del Proyecto: AcadEvalSys

Este archivo es un resumen técnico generado por Gemini para una referencia rápida sobre la estructura, tecnologías y comandos del proyecto.

## 1. Descripción General

**AcadEvalSys** es un sistema de gestión y evaluación académica basado en la web, diseñado con principios de Clean Architecture.

## 2. Stack Tecnológico Principal

- **Backend:** .NET 8, ASP.NET Core, EF Core, MediatR (CQRS)
- **Frontend:** React 19, Tailwind CSS
- **Base de Datos:** PostgreSQL (gestionada con Docker)
- **CI/CD:** GitHub Actions
- **Contenedores:** Docker

## 3. Arquitectura

El proyecto utiliza **Clean Architecture** con la siguiente separación de responsabilidades:

- `AcadEvalSys.Domain`: Entidades, interfaces y lógica de negocio principal.
- `AcadEvalSys.Application`: Lógica de la aplicación, casos de uso (Commands/Queries) y DTOs.
- `AcadEvalSys.Infrastructure`: Implementaciones de acceso a datos (EF Core), servicios externos, etc.
- `AcadEvalSys.WEB.Server`: API web de ASP.NET Core que expone los endpoints.
- `AcadEvalSys.WEB.Client`: Aplicación de usuario final construida en React.
- `tests/`: Proyectos de pruebas unitarias y de integración.

## 4. Comandos Esenciales

### Iniciar Servicios (Base de Datos)
```bash
docker-compose up -d
```

### Ejecutar el Backend
```bash
dotnet run --project src/AcadEvalSys.WEB/AcadEvalSys.WEB.Server/AcadEvalSys.API.csproj
```

### Ejecutar el Frontend (Modo Desarrollo)
```bash
cd src/AcadEvalSys.WEB/AcadEvalSys.WEB.Client
npm install
npm run dev
```

### Compilar la Solución Completa
```bash
dotnet build AcadEvalSys.sln
```

### Ejecutar Todas las Pruebas
```bash
dotnet test
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AcadEval-System)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/AcadEval-System)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
