# TaskManagement

Task Management system — a C# application designed to run against SQL Server for task, project, and user management. This single README combines a concise professional overview with a Functions & Tasks backlog so developers and maintainers can quickly get started, update the DAL (DatabaseAccess), and drive development.

[![Language](https://img.shields.io/badge/Language-C%23-blue)]()
[![DB](https://img.shields.io/badge/Database-SQL%20Server-brightgreen)]()
[![Status](https://img.shields.io/badge/Status-Active-green)]()

---

## Overview

TaskManagement centralizes project and task tracking for small teams and internal tooling. The codebase (primarily C# with some T-SQL) is intended to run with Microsoft SQL Server. The repository includes a Data Access Layer (DAL) where the connection string lives (see DatabaseAccess file, line ~15) — update that to your machine's SQL Server name to connect.

Core goals:
- Reliable CRUD for tasks, projects, users and role-based actions
- Clear DAL separation to make DB provider swaps or ORM adoption straightforward
- Minimal friction for onboarding: SQL scripts to provision DB and a single place to update the Data Source
- Testable services and a CI-friendly build using dotnet CLI

Note (from repo description): To run locally, obtain the data files and run them on SQL Server, then edit the DAL file DatabaseAccess (line ~15) to set the Data Source to your machine's server name — check the "Server name" shown in SSMS when connecting.

---

## Key features

- Task CRUD (create, read, update, delete), search, and filtering
- Projects and milestones with task associations
- Users, roles and simple RBAC (Admin, Manager, User)
- Comments/notes and attachments (optional)
- Import/Export of tasks (CSV/T-SQL seed)
- Basic reporting (per-project progress, overdue tasks)
- Database-first scripts and T-SQL helpers for provisioning
- Clear DAL (DatabaseAccess) wrapper for connections and queries
- Tests and instructions to run against SQL Server

---

## Technology & recommended stack

- Language: C# (≈98%), T-SQL for DB scripts
- Target runtime: .NET 6 / .NET 7 (please confirm which you prefer)
- Data provider: Microsoft SQL Server (local dev: SQL Server Express or Docker)
- Data access: Current repo uses a DatabaseAccess DAL file (ADO.NET style). Optionally migrate to EF Core or Dapper.
- Build: dotnet CLI (dotnet build, dotnet test, dotnet run)
- IDE: Visual Studio / VS Code

---

## System requirements

- .NET SDK 6 or 7 (install the version the repo targets)
- SQL Server (Express/Developer) or Docker image for SQL Server
- Optional: SQL Server Management Studio (SSMS) for DB management
- Recommended: Docker for local DB and CI integration

---

## Quick start — provision DB and run

1. Provision the database
   - If the repo includes a /database or /sql folder, run the provided .sql scripts in order via SSMS or sqlcmd to create the database schema and seed data.
   - Example using sqlcmd:
     ```bash
     sqlcmd -S "YOUR_SERVER_NAME" -i ./database/01_schema.sql -o ./logs/schema.log
     sqlcmd -S "YOUR_SERVER_NAME" -i ./database/02_seed.sql -o ./logs/seed.log
     ```
   - If using EF Core Migrations (repo-dependent):
     ```bash
     dotnet ef database update --project ./src/YourProject.csproj
     ```

2. Update the connection string in DAL
   - Open the DAL file: src/.../DAL/DatabaseAccess.cs (or the path used in this repo).
   - Edit line ~15 where the connection string placeholder exists:
     Example:
     ```csharp
     // DatabaseAccess.cs (around line 15)
     private const string ConnectionString = "Data Source=YOUR_SERVER_NAME;Initial Catalog=TaskManagementDB;Integrated Security=True;";
     ```
   - Replace YOUR_SERVER_NAME with the server shown in SSMS when you connect (e.g., .\SQLEXPRESS or (localdb)\MSSQLLocalDB or your machine name).

3. Build & run the app
   ```bash
   dotnet restore
   dotnet build
   dotnet run --project src/TaskManagement.App/TaskManagement.App.csproj
   ```
   - Adjust project paths to match the repository structure.

4. Troubleshooting connection errors
   - Ensure SQL Server is running and accessible.
   - If using Integrated Security and you get login errors, try specifying SQL credentials:
     ```
     Data Source=YOUR_SERVER_NAME;Initial Catalog=TaskManagementDB;User ID=sa;Password=YourPassword123;
     ```
   - For local dev, (localdb)\MSSQLLocalDB is often available without extra setup.
   - Confirm firewall rules and network access when using remote DB.

---

## Functions & Tasks (combined: professional backlog)

Below is a prioritized (P0/P1/P2) set of functions and implementation tasks with acceptance criteria so you can convert items to issues or implement them immediately.

### Core functions (overview)

1. Task Management
   - Description: Create, list, update, delete tasks with validation, due dates, priority, status.
   - API/UI: Task CRUD endpoints or UI screens (depending on app type).
   - Acceptance:
     - Create -> returns new task with ID
     - Update -> changes persisted; validation enforced (title required)
     - Delete -> soft delete by default
   - Tests: unit tests for service layer; integration tests hitting DB

2. Project Management
   - Description: Group tasks under projects; project-level metadata and progress calculations.
   - Acceptance:
     - Project CRUD + list tasks by project
     - Project progress computed from task statuses

3. Users & Roles
   - Description: Basic users, role assignment, and RBAC checks (Admin, Manager, User).
   - Acceptance:
     - Only Admins can delete projects; Managers can create tasks in their projects

4. DAL & DatabaseAccess
   - Description: Centralized DB connection and helper methods.
   - Acceptance:
     - Single place to configure connection string
     - Safe, disposable use of SqlConnection/SqlCommand; no connection leaks
   - Task: Parameterize or externalize the connection string (appsettings.json or environment variables) instead of hardcoding.

5. Import/Export & DB scripts
   - Description: CSV/T-SQL import for bulk tasks and SQL scripts to provision DB.
   - Acceptance:
     - Import dry-run and error reporting
     - SQL scripts include schema and seed steps

6. Reporting & Exports
   - Description: Per-project reports, overdue tasks, export to CSV/Excel.
   - Acceptance:
     - Export endpoints or buttons produce downloadable CSV

7. Background Jobs & Notifications
   - Description: Periodic jobs (overdue reminders) and optional email notifications.
   - Acceptance:
     - Jobs scheduled and idempotent; email adapter mockable

8. Auditing & Activity Log
   - Description: Track create/update/delete with actor and timestamp.
   - Acceptance:
     - Audit entries persisted and searchable

---

### Suggested data model (high level)

- User { Id, Username, Email, DisplayName, Roles[], CreatedAt, UpdatedAt }
- Project { Id, Name, Description, OwnerUserId, Status, CreatedAt }
- Task { Id, ProjectId, Title, Description, AssignedToUserId, Status, Priority, DueDate, CreatedAt, UpdatedAt, DeletedAt }
- Comment { Id, TaskId, AuthorUserId, Body, CreatedAt }
- Audit { Id, EntityType, EntityId, Action, ActorUserId, Details, Timestamp }

---

### Prioritized implementation tasks (backlog)

P0 — Core (must have)
- Task: Externalize connection string and remove hardcoded Data Source in DatabaseAccess.cs (line ~15)
  - Complexity: low
  - Tests: Confirm application connects using environment variable or appsettings.json
- Task: Add/verify SQL scripts that create schema and seed data
  - Complexity: low
  - Tests: Scripts run successfully on a local SQL Server instance
- Task: Implement Task CRUD + Project CRUD + basic UI or API endpoints
  - Complexity: medium
  - Tests: Unit tests for services; integration tests for DB persistence
- Task: Add basic authentication & role checks
  - Complexity: medium

P1 — Important
- Task: Import/export tools and dry-run validation for bulk data
  - Complexity: medium
- Task: Add audit logging for critical actions
  - Complexity: medium
- Task: Add unit and integration tests, CI pipeline (GitHub Actions)
  - Complexity: low-medium

P2 — Enhancements
- Task: Migrate DAL to EF Core (optional) or add Dapper option
  - Complexity: medium-high
- Task: Add background jobs (reminder emails) and a pluggable notification provider
  - Complexity: medium
- Task: Add Docker Compose sample (SQL Server + app) for dev environment
  - Complexity: low-medium

---

## Example DB connection string snippets

- Integrated Security (Windows auth)
  ```
  Data Source=YOUR_SERVER_NAME;Initial Catalog=TaskManagementDB;Integrated Security=True;
  ```
- SQL Authentication
  ```
  Data Source=YOUR_SERVER_NAME;Initial Catalog=TaskManagementDB;User ID=sa;Password=YourPassword123;
  ```
- Using LocalDB (developer convenience)
  ```
  Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=TaskManagementDB;Integrated Security=True;
  ```

Remember: the repo description specifically points to DatabaseAccess file line ~15 — update that line to point to your machine's Server name (check SSMS -> Server name).

---

## Tests

- Unit tests (xUnit / NUnit / MSTest depending on repo): run via dotnet test
- Integration tests: run against a local SQL Server or Testcontainers SQL Server in CI
- CI recommendation: GitHub Actions workflow that builds, runs unit tests, and (optionally) runs integration tests with a SQL Server service

---

## Docker (optional)

Run SQL Server in Docker for local testing:
```bash
docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=YourPassword123" -p 1433:1433 mcr.microsoft.com/mssql/server:2019-latest
```

---

## Contributing

- Fork, create a feature branch, add tests, and open a PR with clear description and acceptance criteria.
- Add/update a CONTRIBUTING.md with code style and testing expectations.
- Document schema changes and maintain DB migration scripts or EF Migrations.

---

## Troubleshooting & FAQ

- "Cannot open database" — ensure the Initial Catalog exists or check permissions for the SQL user.
- "Login failed for user" — verify credentials and authentication mode (Windows vs SQL).
- "Server not found" — confirm the Data Source matches SSMS server name (including instance name).
- If DatabaseAccess is hardcoded, move the string to appsettings.json or environment variables before committing.

---

## License & maintainers

- License: (replace with MIT / Apache-2.0 / proprietary as desired)
- Maintainer: oggishi (https://github.com/oggishi)
- Contact: add an email or profile link in the repo settings

---

Thank you for using TaskManagement. This README gives a professional overview and a concrete Functions & Tasks backlog so you can quickly provision the DB, update the DAL (DatabaseAccess.cs line ~15), and continue development.
