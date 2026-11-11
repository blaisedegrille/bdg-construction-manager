# Renovation Project Manager – Product Concept and Development Plan

## 1. Product goal

Build a desktop application for planning and tracking multiple renovation projects. Users should be able to see what needs to happen next, who is responsible, when work is due, how much it costs, how progress compares with the plan, and where related documents are stored.

Workflows and their history are the core of the product. A Jira-style board is a useful view, but tasks should remain connected to renovation stages, milestones, contractors, budgets, costs, and documents.

## 2. Recommended technology

- **Electron** desktop client for Windows, macOS, and Linux.
- **React + TypeScript** for the user interface.
- **MUI** for UI components and light/dark themes.
- **Redux Toolkit + RTK Query** for application state and API data/cache management; **React Hook Form + Zod** for forms and validation.
- **NestJS** modular API for business logic, authentication, authorization, and data access.
- **PostgreSQL** as the primary database, running in Docker.
- **Docker Compose** for starting the API, database, and optional file-storage service locally or on a server.
- **Prisma** for data modeling and database migrations.

### Architecture

Use a client–server architecture: the React interface runs in an Electron desktop client, while the NestJS API and PostgreSQL run in Docker containers. The containers can run on the user's computer or on an accessible server. The client connects to a configurable API URL. Use Electron preload/IPC for operating-system integrations; keep Node.js access out of the renderer and enable context isolation.

Because the backend is a shared API, user accounts, project membership, and authorization should be part of the foundational model. Docker Compose should support development and self-hosted deployment. Managed cloud hosting, remote access hardening, monitoring, and automatic updates can be added later. Offline operation is outside the initial release; if required, it needs a dedicated synchronization design.

### Recommendation and alternatives

**Backend recommendation:** keep **NestJS + PostgreSQL + Prisma + Docker Compose**. NestJS fits the TypeScript-based React/Electron stack, its modular structure suits separate auth, projects, tasks, costs, and documents domains, and its API can later serve a browser client. React does not require a different backend; the client and server communicate through an HTTP API as separate processes.

**Recommended full stack:** Electron + React + TypeScript + Vite + MUI + Redux Toolkit/RTK Query + React Hook Form/Zod; Electron Forge for packaging; NestJS REST API, PostgreSQL, and Prisma in Docker Compose. Use RTK Query for server-data caching and updates, and Redux slices for shared client state such as authentication, active project, and UI preferences. Keep transient form values out of the global store. Verify the Forge/Vite integration in the initial prototype because Forge currently documents its Vite plugin as experimental. REST is enough initially; add real-time updates only if a concrete need emerges. The first technical milestone should verify API connectivity, migrations, persistent storage, file attachments, and starting the complete stack with one documented command.

Fastify on its own is a leaner alternative, but leaves more architectural conventions to the project. ASP.NET Core is a strong option if the team prefers C# or Microsoft tooling; NestJS keeps this TypeScript-centered project on one language across client and server. The React interface can also be offered in a browser later.

## 3. Main user journeys

1. Create a project with a name, address/location, type, planned start and finish dates, and total budget.
2. Break it into stages such as planning, demolition, plumbing, electrical, flooring, painting, and handover, or start from a template.
3. Add tasks and milestones with an owner, due date, status, dependencies, estimated duration, and planned cost.
4. Record status changes, notes, actual time, actual cost, and related photos or documents as work progresses.
5. Review planned versus actual progress and spending, and identify overdue work or items at risk of exceeding budget.
6. Archive completed projects while keeping them searchable.

## 4. Features and views

### Projects

- Create, edit, search, and archive multiple projects.
- Project details: description, location, dates, budget, status, and members.
- Starter templates for common renovation types; custom templates can follow later.
- Project switcher and recently opened projects.

### Planning and workflows

- Stage and task list plus a Kanban board (for example: To do, In progress, Blocked, Done).
- Task details: title, description, status, priority, assignee, dates, deadline, estimated/actual time, and planned/actual cost.
- Milestones and task dependencies; calendar or timeline views can follow later.
- Checklists, comments, activity history, and attachments.
- Filters by status, owner, stage, and due date.

### Costs

- Budget categories and estimates by task or stage (materials, labor, permits, delivery, other).
- Track planned, committed, and actually paid amounts separately.
- Cost entries with date, category, vendor, note, and optional task link.
- Compare budget and actual spending, including amount and percentage variance.
- Configure currency and tax at project level. Full accounting is not an MVP goal.

### Time and progress

- Planned and actual start/finish dates, estimated and actual effort.
- Task and stage progress, plus milestone completion.
- Activity history showing what changed and when.
- Manual time entry in the MVP; an optional timer can be added later.

### Documents and photos

- Attach quotes, invoices, contracts, plans, and before/after photos.
- Store filename, type, upload date, note, and project/task/cost association.
- Store files in a project-organized Docker volume or S3-compatible storage, separate from the database and backup-able independently.
- Define file size limits, supported formats, and deletion behavior.
- Automatic extraction of invoice data can be an optional later feature.

### Users and appearance

- User accounts, sign-in, and project roles: owner, editor, and viewer.
- Light/dark themes, follow-system option, and navigation for Overview, Projects, Tasks, Costs, Documents, and Settings.
- Hungarian interface first; keep strings ready for localization.
- Keyboard navigation, visible focus, and sufficient contrast.

## 5. MVP scope

The first usable release should include:

- Multiple projects with create, edit, and archive actions.
- Project overview with key details and budget.
- Stages, milestones, tasks, and list/Kanban views.
- Status, deadlines, assignees, estimated/actual time, and cost.
- Cost entries and planned-versus-actual summaries.
- Comments and activity history.
- Document attachment, opening, and association with a project or task.
- Light/dark theme, navigation, basic search, and filters.
- Sign-in, project membership, and basic roles.
- Docker Compose setup for the NestJS API, PostgreSQL, and persistent storage.
- Database migrations and documented backup/restore procedures.

Keep real-time collaboration, mobile apps, advanced permission administration, invoice recognition, chat, critical-path Gantt planning, billing, and accounting integrations out of the MVP. Revisit them after user feedback.

## 6. Development roadmap

Estimates are rough, assuming one developer after scope clarification. The technical prototype may change them.

### Phase 0 – product and technical foundation (2–4 days)

- Define user stories, MVP boundaries, and domain terminology.
- Map relationships among projects, stages, tasks, milestones, costs, documents, and users.
- Sketch project list, overview, Kanban, task details, costs, and settings screens.
- Record decisions about hosting, access, backups, and whether offline use is required.

**Deliverable:** agreed MVP specification and screen sketches.

### Phase 1 – scaffolding and technical proof (3–5 days)

- Set up the repository, Electron Forge + Vite + React/TypeScript shell, and MUI theme.
- Configure the Redux Toolkit store, RTK Query API layer, React Hook Form, and Zod.
- Add the modular NestJS API and client-to-API communication.
- Configure Docker Compose with PostgreSQL, environment configuration, and a persistent volume.
- Add the first database migration, health check, and basic logging.
- Prototype persistent file storage and outline backup/restore.
- Verify that the full stack starts and that a packaged client can connect to it.

**Deliverable:** working client plus Dockerized API/database with verified persistent data.

### Phase 2 – application shell and projects (1 week)

- Main navigation, project list, and create/edit/archive flows.
- Core data model and CRUD operations.
- Light/dark theme and settings.
- Empty states, error handling, and confirmation dialogs.

**Deliverable:** a navigable application that manages multiple projects.

### Phase 3 – planning, tasks, and milestones (1–2 weeks)

- Manage project stages and tasks.
- List and Kanban views, task editing, and status changes.
- Deadlines, priority, estimated effort, assignees, and basic dependencies.
- Milestones and project activity history.

**Deliverable:** a project workflow that can be tracked from planning to completion.

### Phase 4 – costs and time tracking (1 week)

- Cost categories and cost entries.
- Record planned, committed, and actual amounts.
- Capture estimated/actual effort and dates.
- Overview cards and simple planned-versus-actual charts/summaries.

**Deliverable:** project cost and time variances are visible.

### Phase 5 – documents and data safety (1 week)

- Attach and open files; associate them with projects, tasks, or costs.
- Configure Docker volume or S3-compatible storage and file deletion rules.
- Export project data and implement backup/restore.
- Complete account and basic role flows if not finished earlier.

**Deliverable:** quotes, invoices, and work photos are stored with the project.

### Phase 6 – polish and first release (3–5 days)

- Empty, loading, and error states; search and filtering.
- Hungarian labels, keyboard use, and theme review.
- Package the desktop client for target platforms; document versioning and updates.
- Try the application with a real renovation example and fix the issues found.

**Deliverable:** a usable client–server release that can be operated with Docker.

### Phase 7 – expanded collaboration and operations (plan separately)

- Project invitations, richer roles, and notifications.
- Remote access, TLS, secret management, and a secure self-hosting guide.
- Optional S3-compatible document storage, automated backups, and restore drills.
- Managed hosting, monitoring, and deployment/update workflows.

**Deliverable:** a reliable shared service accessible remotely.

## 7. Suggested navigation

- **Overview** – all projects, upcoming deadlines, and recent activity.
- **Projects** – project list, creation, and archived projects.
- **Tasks** – task list/board for the selected project.
- **Costs** – budget, categories, and actual spending.
- **Documents** – searchable project documents.
- **Settings** – profile, theme, backups, language, and app settings.

Within a project, use tabs or a sub-navigation for Overview, Plan/Tasks, Milestones, Costs, Documents, and Activity.

## 8. Initial data model

- `User`: account, authentication reference, profile, and status.
- `Project`: name, description, location, status, dates, currency, budget, and creator.
- `ProjectMember`: project, user, and role.
- `Stage`: project, name, order, and planned period.
- `Task`: stage, title/description, status, assignee, priority, deadlines, planned/actual time and cost.
- `Milestone`: project, name, target date, and completion status.
- `TaskDependency`: task and prerequisite relationship.
- `CostEntry`: project/task, category, vendor, amount, currency, date, and status.
- `Attachment`: storage key/path, filename, type, size, and related record.
- `Activity`: user, action, timestamp, and affected record.

## 9. Quality and operational requirements

- Protect PostgreSQL data with persistent Docker volumes and a clear backup/restore process.
- Use stable migrations so upgrades preserve existing projects.
- Keep large file binaries out of the relational database; manage attachments separately.
- Safely handle user-supplied filenames and storage paths.
- Enforce authentication, authorization, and project-level data isolation in the API.
- Keep secrets and persistent data out of container images; provide them through configuration and volumes.
- For remote access, require HTTPS, secure password handling, and regular backups.
- Use strict Electron security settings: context isolation, sandboxing, restricted IPC, a Content Security Policy, and disabled Node integration in the renderer. Validate IPC messages and expose only a narrow, purpose-built API from preload.

## 10. Open product decisions

Resolve these during the prototype and early user feedback:

1. Will the Docker stack run only on one computer, on a local network, or on an internet-accessible server?
2. Which operating systems must the first release support?
3. Should documents live on a Docker volume or in S3-compatible storage?
4. How often should automatic backups run, and where should they be stored?
5. Should currency and tax settings be project-level?
6. Is import from an existing renovation spreadsheet or workflow needed?

**Recommended starting assumption:** a NestJS + PostgreSQL service started with Docker Compose, with a React/Electron client, multiple projects, and basic user accounts. The first release can run on one machine or a self-hosted server; internet-facing access should be an explicit deployment configuration.
