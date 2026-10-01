PROJECT PLAN: Lightweight Project Management App (Plane.so / Jira-style) – .NET 9 Edition
0. Instructions for the implementing AI

    Build this step by step, in the phases listed in section 10. Do not skip tests.

    All code, comments, docblocks, commit messages, UI text, and seed data MUST be in English.

    Follow SOLID, DRY, KISS, and Clean Architecture principles.

    Every service/handler method gets a concise XML doc comment explaining WHY it exists.

    Stack changes are locked to .NET 9 + Blazor.

1. Tech Stack

    Framework & Runtime: .NET 9 (C# 13)

    UI / Frontend: Blazor Web App (.NET 9 Interactive Server or Auto mode), Tailwind CSS v3/v4 (via npm/standalone CLI), MudBlazor or Lucide Icons for Blazor.

    Backend / Architecture: Clean Architecture (Domain, Application, Infrastructure, Web/UI).

    CQRS & Business Logic: MediatR (Queries, Commands, Handlers) + FluentValidation.

    Database & ORM: Entity Framework Core 9 (SQLite for local/test, SQL Server/PostgreSQL ready).

    Authentication: ASP.NET Core Identity (Session/Cookie-based auth with custom Claims for Project Permissions).

    Testing: xUnit / Pest-like fluent assertions (FluentAssertions) + bUnit for Blazor component tests + EF Core In-Memory / SQLite In-Memory.

    Code Quality: C# Nullable Reference Types (<Nullable>enable</Nullable>), StyleCop / Roslyn Analyzers.

2. Core Features (MVP)

    Authentication & Authorization: Individual user accounts (Cookie auth), custom Policy-based authorization for project isolation.

    Projects: List, create, edit, archive projects; manage project members & roles (Admin, Member, Viewer).

    Tasks (Issues): Jira-like properties, auto-generated human-readable IDs (MMI-12), sequential numbering per project.

    Workflow Statuses: Customizable per-project columns/queues with ordering position and categories (Backlog, ToDo, InProgress, InReview, Done).

    Kanban Board & List View: Interactive Blazor Kanban board with Drag-and-Drop between statuses (MudBlazor or custom JS Interop / HTML5 Drag and Drop) + filterable List view.

    Comments & Activity Log: Markdown-supported comments, automated Activity Audit Log created via EF Core Interceptors or Domain Events.

    Wiki Engine: Global company wiki + project-scoped wiki with tree structure and Markdown renderer.

    Automated Seeding: dotnet run --seed or automatic startup DB seeding generating test users, MMI project, and realistic epic/subtasks.

3. Domain Model (C# Entities)
ApplicationUser : IdentityUser<Guid>

    Id (Guid), Name (string), Email (string)

Project

    Id (Guid), Name (string), Key (string, e.g., "MMI"), Description (string), LeadId (Guid), IsArchived (bool), CreatedAt (DateTimeOffset)

    Navigation: Members, Statuses, Tasks, WikiPages

    Rule: Users can query only projects where ProjectMembers.Any(pm => pm.UserId == currentUserId).

ProjectMember (Join entity)

    ProjectId (Guid), UserId (Guid), Role (Enum: Admin, Member, Viewer)

Status

    Id (Guid), ProjectId (Guid), Name (string), Position (int), Category (Enum: Backlog, ToDo, InProgress, InReview, Done), ColorHex (string)

TaskItem (Issue)

    Id (Guid), ProjectId (Guid), Number (int, sequential per project), StatusId (Guid), Title (string), Description (string, Markdown), Type (Enum: Task, Bug, Story, Epic), Priority (Enum: Lowest, Low, Medium, High, Highest), ReporterId (Guid), AssigneeId (Guid?), ParentId (Guid?), DueDate (DateTime?), Position (int), IsDeleted (bool)

    Computed Key Property: public string IssueKey => $"{Project.Key}-{Number}"; (e.g. MMI-12).

Label & TaskLabel

    Label: Id (Guid), ProjectId (Guid), Name (string), ColorHex (string)

    TaskLabel: TaskId (Guid), LabelId (Guid)

Comment

    Id (Guid), TaskId (Guid), UserId (Guid), Body (string, Markdown), CreatedAt (DateTimeOffset), IsDeleted (bool)

ActivityLog

    Id (Guid), TaskId (Guid), UserId (Guid), Event (string), OldValue (string?), NewValue (string?), Timestamp (DateTimeOffset)

WikiPage

    Id (Guid), ProjectId (Guid?), ParentId (Guid?), Title (string), Slug (string), Content (string, Markdown), CreatedById (Guid), UpdatedAt (DateTimeOffset)

4. Authorization & Security (.NET Policies)

    Custom Requirement: ProjectMemberRequirement(ProjectRole minimumRole)

    Global EF Core Query Filter: Automatically filter Projects and TaskItems using current user claims (Project::VisibleTo(userId)).

    Route Authorization: Custom Blazor [Authorize] attributes or Navigation Guards ensuring 403/404 if a non-member tries to navigate to /projects/{key}/*.

5. UI Layout & Blazor Pages

    Layout: MainLayout.razor with persistent left Sidebar (Project switcher, Global Wiki link) and Topbar (Search, User menu).

    /projects – Project Cards Overview.

    /projects/{Key}/board – Blazor Interactive Kanban Board (Drag-and-Drop between status columns).

    /projects/{Key}/tasks – Filterable DataGrid/List (Search, Status, Assignee, Priority filters).

    /projects/{Key}/tasks/{IssueKey} – Detailed Issue View (Title inline edit, status dropdown, assignee selector, subtask list, comments, and activity timeline).

    /projects/{Key}/settings – Project info, Members & Roles management, Custom Statuses.

    /projects/{Key}/wiki & /wiki – Tree navigation + Markdown view/editor.

6. Seed Data (DbInitializer.cs)
Users (password = Password123!)

    wojciech@test.ie (Wojciech Mruk)

    daren@test.ie (Daren)

    derek@test.ie (Derek)

Seeded Project

    Name: "Mill Midland of Ireland", Key: MMI

    Lead: Wojciech (Admin), Daren (Member), Derek (Member)

    Labels: database, infrastructure, migration

Seeded Tickets for MMI (Epic & Subtasks)

    EPIC MMI-1: "Migrate data from Mill Midland of Ireland's server to a dedicated server"

        MMI-2 (Task, High): "Investigate access to the client's database"

        MMI-3 (Task, Medium): "Determine data pull frequency"

        MMI-4 (Task, High): "Set up a local in-house server replicating the client's server"

        MMI-5 (Task, High): "Set up the target dedicated server"

        MMI-6 (Task, Medium): "Identify required tables on the target server"

        MMI-7 (Story, High): "Write stored procedure to perform the data migration"

        MMI-8 (Task, Medium): "Configure scheduler on the dev server"

        MMI-9 (Task, Low): "Write migration runbook in project wiki"

7. Delivery Phases (.NET 9 Implementation Roadmap)

    Phase 1: Project Solution Scaffold & Auth

        Create Blazor Web App solution (.Core, .Application, .Infrastructure, .Web).

        Setup ASP.NET Core Identity & Tailwind CSS.

    Phase 2: EF Core Migration & Domain Entities

        Setup EF Core DB Context, Fluent API configurations, indexes, soft delete filters.

    Phase 3: Database Seeder & Unit/Integration Tests Scaffolding

        Implement DbInitializer with MMI project and tickets + Pest/xUnit tests.

    Phase 4: Project Management & Settings UI

        Projects CRUD, Membership role assignment, and Workflow Status settings.

    Phase 5: Task Management & Issue Key Generator

        Task creation, thread-safe sequence number generator per project, filtering pipeline.

    Phase 6: Blazor Interactive Kanban Board

        Implement Kanban board with drag-and-drop status update commands.

    Phase 7: Comments & Activity Log Interceptor

        Add comment thread and automated EF Core Interceptor to write audit activity logs on entity changes.

    Phase 8: Wiki Engine

        Implement Global and Project-level Wiki pages with Markdown rendering (Markdig).

    Phase 9: Polish & Static Analysis

        UI polish, Roslyn analysis cleanup, README setup commands.
