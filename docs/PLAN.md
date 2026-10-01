# ICM Projects — Implementation Plan

## 0. Decisions

- **Single project**: one `PMApp.csproj` (Blazor Web App, .NET 9) containing all layers in **folders** —
  `Domain/`, `Application/`, `Infrastructure/`, `Components/` (Web). NO separate class library projects
  (this corrects the earlier multi-project split used in icm-knowledge).
  Namespace layout stays Clean-Architecture style (`ICM.Domain`, `ICM.Application`, `ICM.Infrastructure`,
  `ICM.Web`) while shipping as one assembly.
- **Stack (locked)**: .NET 9 / C# 13, Blazor Interactive Server, EF Core 9 + SQLite, ASP.NET Core Identity,
  Tailwind CSS (standalone CLI), Markdig, xUnit + FluentAssertions (+ bUnit where UI logic warrants).
- **UI reference**: mockups in `docs/project.html` (Kanban board) and `docs/ticket.html` (issue detail view)
  define the visual target — Tailwind utility classes, Jira-like layout, light/dark via `darkMode: 'class'`.
- **Language**: all code, comments, commit messages, UI text and seed data in English.
- **Process**: every phase = one Jira Epic; every work item = Story/Task linked to its Epic.
  Tests are not skippable; each phase defines its acceptance criteria.

## 1. Phases (Jira Epics)

| # | Phase | Scope summary |
|---|-------|---------------|
| 1 | Phase 1: Project Scaffold & Auth | Single csproj, .NET 9 Blazor, Identity login/register, Tailwind pipeline, app shell (MainLayout, sidebar, topbar) |
| 2 | Phase 2: Domain & Persistence | Entities (Project, TaskItem, Status, Label, Comment, WikiPage, ActivityLog), EF Core DbContext, Fluent API, soft-delete, migrations, SQLite |
| 3 | Phase 3: Seeder & Testing Infrastructure | DbInitializer (users, MMI project, epic + subtasks MMI-1..9), xUnit/FluentAssertions scaffolding, test DB strategy |
| 4 | Phase 4: Project Management & Settings UI | Projects list/create/edit/archive, members & roles (Admin/Member/Viewer), per-project workflow statuses, authorization policies |
| 5 | Phase 5: Task Management & Issue Keys | Task CRUD, thread-safe sequential issue keys (MMI-12), filters (status/assignee/priority/type/search), list view, issue detail view |
| 6 | Phase 6: Kanban Board | Interactive board, drag-and-drop between status columns, optimistic update + command, match docs/project.html mockup |
| 7 | Phase 7: Comments & Activity Log | Markdown comments (Markdig), EF Core SaveChanges interceptor writing ActivityLog entries, activity timeline in issue view |
| 8 | Phase 8: Wiki Engine | Global + project wiki, tree structure, Markdown render/edit, slug routing |
| 9 | Phase 9: Polish & Static Analysis | UI polish per mockups, analyzer cleanup, nullable enabled, README, final test pass |

## 2. Cross-cutting rules

- CQRS-style handlers (Command/Query + Handler) in `Application/` — no MediatR dependency unless needed;
  keep it simple (KISS) with explicit interfaces.
- Authorization: ASP.NET Core policy-based (`ProjectMemberRequirement` with minimum role) + EF Core global
  query filters scoping Projects/TaskItems to the current user's memberships.
- Issue key generation: per-project sequential `Number` computed under transaction/unique index
  (`(ProjectId, Number)` unique) — no race conditions.
- Soft delete: `IsDeleted` + global query filters; audit fields `CreatedAt`/`UpdatedAt`.
- Validation: FluentValidation for commands; domain invariants enforced in entities.
- Every handler/service method gets a concise XML doc comment explaining WHY it exists.

## 3. Definition of Done (per story)

- [ ] Code matches mockups where UI is involved (docs/*.html)
- [ ] Unit tests for handlers/domain rules (xUnit + FluentAssertions), all green
- [ ] Nullable reference types enabled, no analyzer warnings introduced
- [ ] XML doc comments on public handlers/services
- [ ] Migration created (when schema changed) and DB seeds correctly
- [ ] Manual check against the phase's acceptance criteria
