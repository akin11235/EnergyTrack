Excellent question! Let me show you what **actually happens** in real companies (not the idealized textbook version).

---

## The Real Process (Most Companies)

### **Phase 0: Before Anyone Writes Code** (1-2 weeks)

#### **The Kickoff Meeting**
```
Present in room:
- Product Manager: "We need to track energy usage for clients"
- Engineering Manager: "How soon? What's the budget?"
- Tech Lead: "What's our timeline?"
- Senior Dev: "Do we have design mockups?"
- Designer: "Working on it"
- DevOps: "What infrastructure?"
```

**Outcome:** 
- Project goals defined (often vague at first)
- Timeline estimate (usually gets doubled)
- Team assigned

---

#### **Discovery Phase - Key Questions**

**Product Manager asks:**
- Who are the users? (Consultants? Clients? Both?)
- What problems are we solving? (Priority order)
- What's the MVP? (Minimum Viable Product)
- What can wait for v2?

**Tech Lead asks:**
- What's our tech stack? (Already decided: .NET Core API + React frontend)
- Hosting? (Azure? AWS? On-premise?)
- Database? (SQL Server, PostgreSQL?)
- Auth? (SSO with Azure AD? Custom?)
- Performance requirements? (100 users? 10,000 users?)

**Everyone together:**
```
Meeting transcript:

PM: "We need all buildings to show real-time energy usage"
Dev: "Define 'real-time'"
PM: "Like... updated frequently"
Dev: "Every second? Minute? Hour? Daily?"
PM: "Daily is fine"
Dev: "So not real-time. Got it."

PM: "Users need to upload utility bills"
Dev: "File size limits?"
PM: "I don't know... normal bill size?"
Dev: "I'll assume 10MB max"
Designer: "Should we support multiple files?"
PM: "...I'll ask the stakeholders"
```

---

### **Phase 1: Technical Design Document (TDD)** (3-5 days)

Senior dev or Tech Lead writes this. Here's what it looks like:

```markdown
# EnergyTrack - Technical Design Document

## 1. System Architecture

### High-Level Components
┌─────────────┐      ┌──────────────┐      ┌──────────┐
│   React     │─────▶│   .NET API   │─────▶│   SQL    │
│   Frontend  │      │   (REST)     │      │  Server  │
└─────────────┘      └──────────────┘      └──────────┘
                            │
                            ▼
                     ┌──────────────┐
                     │  Azure Blob  │
                     │   Storage    │
                     └──────────────┘

### Tech Stack Decision Matrix

| Option | Pros | Cons | Decision |
|--------|------|------|----------|
| SQL Server | Team knows it, good tooling | Expensive | ✅ Chosen |
| PostgreSQL | Free, great features | Team learning curve | ❌ |
| MongoDB | Flexible schema | No joins, team unfamiliar | ❌ |

### Database Choice: SQL Server
**Why:** 
- Team expertise
- Strong relationship modeling (Client → Buildings → Meters → Readings)
- Transaction support needed for audits
- Company already has licenses

## 2. Database Schema

[Entity diagrams we discussed earlier]

## 3. API Design

### Authentication
- JWT tokens (24hr expiry)
- Refresh tokens (7 day expiry)
- Azure AD integration (Phase 2)

### Authorization
- Role-based (enum: SystemAdmin, Consultant, ClientAdmin, ClientUser)
- Resource-based (ClientUser can only see their client's buildings)

## 4. Key Technical Decisions

### File Storage
**Decision:** Azure Blob Storage
**Alternatives Considered:**
- Local filesystem (not scalable)
- Database (too large, performance issues)
**Rationale:** Scalable, separate concerns, easy CDN integration

### Pagination
**Decision:** Offset-based (page/size parameters)
**Future:** Consider cursor-based for large datasets

### Caching Strategy
**Phase 1:** None (optimize later)
**Phase 2:** Redis for emissions calculations (expensive to compute)

## 5. Non-Functional Requirements

| Requirement | Target | How We'll Achieve |
|-------------|--------|-------------------|
| Response Time | <500ms for 95% of requests | Indexed queries, pagination |
| Uptime | 99.5% | Azure App Service, health checks |
| Concurrent Users | 200 | Async/await, connection pooling |
| Data Retention | 10 years | Partitioned tables by year |

## 6. Security Considerations

- [ ] All endpoints require authentication (except login/register)
- [ ] HTTPS only
- [ ] SQL injection prevention (EF Core parameterization)
- [ ] File upload validation (type, size)
- [ ] Rate limiting (100 req/min per user)
- [ ] Audit logging (who changed what, when)

## 7. Deployment Architecture

**Environments:**
- DEV: Developer local + shared dev server
- QA: Testing environment (auto-deploy from main branch)
- STAGING: Production clone (manual deploy for demos)
- PRODUCTION: Live system (manual deploy, requires approval)

## 8. Open Questions

- How do we handle timezone conversions? (Buildings in different zones)
- What's the data migration plan from old Excel system?
- Do we need offline mode for field audits?
- What's the disaster recovery plan?

## 9. Out of Scope (Not in MVP)

- Mobile app
- Advanced analytics / ML predictions
- Integration with utility company APIs
- Multi-language support
- Custom branding per client
```

**This document gets reviewed** by:
- Engineering team (technical feasibility)
- Product (does it solve the problem?)
- DevOps (can we deploy/maintain this?)
- Security team (any red flags?)

---

### **Phase 2: Break Into User Stories** (1 day)

Product Manager + Tech Lead work together:

```
Epic: Building Management
├── Story 1: As a consultant, I can add a new building
│   ├── Task: Create Building model
│   ├── Task: Create BuildingRepository
│   ├── Task: Create BuildingService
│   ├── Task: Create POST /api/buildings endpoint
│   ├── Task: Add validation
│   └── Task: Write integration tests
│   Points: 5
│
├── Story 2: As a consultant, I can view all buildings for a client
│   ├── Task: Create GET /api/clients/{id}/buildings endpoint
│   ├── Task: Add pagination
│   ├── Task: Add search/filter
│   └── Task: Add tests
│   Points: 3
│
└── Story 3: As a client admin, I can only see my company's buildings
    ├── Task: Implement authorization policy
    ├── Task: Add ClaimsPrincipal checks
    ├── Task: Add tests for different roles
    └── Task: Update Swagger docs
    Points: 8
```

**Estimation Meeting (Planning Poker):**
```
PM: "How hard is Story 1?"
Dev 1: "3 points" (shows card)
Dev 2: "8 points" (shows card)
Dev 1: "Why 8??"
Dev 2: "We need to add address validation, duplicate checking..."
Dev 1: "Oh right. I change to 5"
```

---

### **Phase 3: Sprint Planning** (2-4 weeks per sprint)

**Sprint 1 Goals** (Week 1-2):
```
Must Have:
✅ Project setup (repos, CI/CD pipeline)
✅ Database schema + migrations
✅ User authentication (JWT)
✅ Building CRUD endpoints
✅ Basic integration tests

Nice to Have:
⚠️ Meter CRUD endpoints

Stretch:
❌ File upload (move to Sprint 2)
```

**Team Capacity:**
- 2 senior devs × 2 weeks = 40 points
- 1 mid-level dev × 2 weeks = 15 points
- Total: 55 points
- Commitment: 45 points (buffer for bugs/meetings)

---

## What Actually Happens Day 1

### **Day 1 Morning - Project Setup**

**Tech Lead does:**
```bash
# Create solution structure
dotnet new sln -n EnergyTrack

dotnet new webapi -n EnergyTrack.Api
dotnet new classlib -n EnergyTrack.Core      # Domain models
dotnet new classlib -n EnergyTrack.Infrastructure  # Data access
dotnet new xunit -n EnergyTrack.Tests

dotnet sln add EnergyTrack.Api
dotnet sln add EnergyTrack.Core
dotnet sln add EnergyTrack.Infrastructure
dotnet sln add EnergyTrack.Tests

# Setup Git
git init
git add .
git commit -m "Initial project structure"
git remote add origin https://github.com/metergy/energytrack.git
git push -u origin main
```

**Creates folder structure:**
```
EnergyTrack/
├── src/
│   ├── EnergyTrack.Api/
│   │   ├── Controllers/
│   │   ├── Middleware/
│   │   └── Program.cs
│   ├── EnergyTrack.Core/
│   │   ├── Entities/
│   │   ├── Interfaces/
│   │   ├── DTOs/
│   │   └── Exceptions/
│   └── EnergyTrack.Infrastructure/
│       ├── Data/
│       ├── Repositories/
│       └── Services/
├── tests/
│   └── EnergyTrack.Tests/
├── docs/
│   ├── TDD.md
│   └── API.md
└── .github/
    └── workflows/
        └── ci.yml
```

**Sets up CI/CD (GitHub Actions):**
```yaml
# .github/workflows/ci.yml
name: CI Build

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 8.0.x
    - name: Restore
      run: dotnet restore
    - name: Build
      run: dotnet build --no-restore
    - name: Test
      run: dotnet test --no-build --verbosity normal
```

---

### **Day 1 Afternoon - Team Divides Work**

**Daily Standup (15 min):**
```
Tech Lead: "Okay, who wants what?"

Senior Dev 1: "I'll do authentication + user management"
Senior Dev 2: "I'll set up the database and entities"
Mid Dev: "I'll start on Building CRUD once DB is ready"
DevOps: "I'll get Azure resources provisioned"

Tech Lead: "Good. Let's sync EOD on any blockers"
```

---

### **Day 2-10 - Actual Development**

#### **Common Team Practices:**

**1. Branch Strategy (Git Flow)**
```bash
main          # Production
  └── develop     # Integration branch
      ├── feature/auth-jwt          # Senior Dev 1
      ├── feature/building-crud     # Mid Dev
      └── feature/database-schema   # Senior Dev 2
```

**2. Pull Request Process**
```
Developer writes code
    ↓
Creates PR
    ↓
Automated checks run (build, tests, linting)
    ↓
Code review (at least 1 approval required)
    ↓
Comments/changes requested
    ↓
Developer fixes
    ↓
Approved & merged to develop
```

**Real PR Comment Thread:**
```
Reviewer: "Why aren't you using the repository pattern here?"
Developer: "This is just a helper method, seemed overkill"
Reviewer: "Let's stay consistent. Can cause confusion later"
Developer: "Fair point. Updated"

Reviewer: "Did you add tests for the error cases?"
Developer: "Oops, forgot. Adding now"

Reviewer: "Nice work on the logging! 👍"
```

**3. Daily Standups (Every morning, 15 min)**
```
Each dev answers:
1. What did I do yesterday?
   "Finished auth endpoints, starting on refresh tokens"

2. What am I doing today?
   "Will complete refresh token logic and add tests"

3. Any blockers?
   "Waiting for DevOps to provision Azure AD app registration"
```

**4. Code Review Standards**
```
Checklist before approving PR:
□ Code follows team conventions
□ Tests added for new functionality
□ No hardcoded secrets
□ Error handling in place
□ Logging added
□ Documentation updated
□ No TODO comments left
□ Performance considerations noted
```

---

## Real Challenges That Come Up

### **Week 1 Issues:**

**Problem 1: Design Changes**
```
Designer (Slack message): "Hey, client wants to track water usage too, 
not just electric and gas. Can we add that?"

Dev: "Sure, but that changes the data model"
PM: "How long?"
Dev: "2 hours if we do it now, 2 days if we do it later"
PM: "Do it now"

[Developer updates UtilityType enum, reruns migration]
```

**Problem 2: Unclear Requirements**
```
Dev: "@PM what does 'building type' mean? Office, Warehouse...?"
PM: "Let me check with stakeholders"
[3 hours later]
PM: "Here's the list: Office, Retail, Industrial, Healthcare, 
     Education, Multi-Family, Other"
Dev: "Adding to enum. Thanks!"
```

**Problem 3: Environment Setup**
```
Junior Dev: "My local database won't connect"
Senior Dev: "Did you update your appsettings.Development.json?"
Junior Dev: "There's a Development one??"
Senior Dev: [Shares template, helps debug for 30 min]
```

**Problem 4: Dependency Conflicts**
```
Dev 1: "I'm getting build errors after pulling latest"
Dev 2: "I upgraded Entity Framework yesterday"
Dev 1: "Can you Slack the migration commands?"
Dev 2: [Posts: dotnet ef database update]
```

---

## Sprint Ceremonies

### **1. Sprint Planning (Start of sprint)**
- Review backlog
- Commit to stories for this sprint
- Break down large stories
- Assign points

### **2. Daily Standups**
- Quick sync
- Identify blockers
- 15 minutes max

### **3. Mid-Sprint Check-in**
- Are we on track?
- Adjust scope if needed
- Demo to stakeholders (informal)

### **4. Sprint Review/Demo (End of sprint)**
```
PM demos to stakeholders:
"Here's what we built this sprint..."
[Shows working features in QA environment]

Stakeholders: "Looks good! Can we add...?"
PM: "Let's add that to the backlog for next sprint"
```

### **5. Retrospective**
```
What went well:
✅ Good collaboration on the database schema
✅ CI/CD pipeline saved us time
✅ Code reviews caught several bugs early

What didn't go well:
❌ Underestimated file upload complexity
❌ Too many meetings interrupted focus time
❌ DevOps bottleneck on environment setup

Action items for next sprint:
→ Block "focus time" on calendars (no meetings)
→ DevOps to document environment setup
→ Add file upload to Sprint 2 with better estimate
```

---

## Documentation That Gets Created

### **1. README.md**
```markdown
# EnergyTrack

## Getting Started

### Prerequisites
- .NET 8 SDK
- SQL Server 2019+
- Visual Studio 2022 or Rider

### Setup
1. Clone repo: `git clone...`
2. Update appsettings: Copy appsettings.Development.template.json
3. Run migrations: `dotnet ef database update`
4. Run: `dotnet run --project EnergyTrack.Api`

### Running Tests
`dotnet test`

## Architecture
See docs/TDD.md

## API Documentation
Swagger: https://localhost:5001/swagger
```

### **2. API Documentation (Swagger + Manual)**
Auto-generated from attributes:
```csharp
/// <summary>
/// Creates a new building for a client
/// </summary>
/// <param name="clientId">The client ID</param>
/// <param name="dto">Building details</param>
/// <returns>Created building</returns>
/// <response code="201">Building created successfully</response>
/// <response code="400">Invalid input</response>
/// <response code="404">Client not found</response>
[HttpPost]
[ProducesResponseType(typeof(BuildingDto), StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
```

### **3. Runbook (For Operations)**
```markdown
## Deployment Process
1. Merge PR to main
2. Tag release: `git tag v1.0.0`
3. CI builds and creates artifacts
4. Manual approval in Azure DevOps
5. Deploy to Production
6. Smoke tests
7. Monitor logs for 1 hour

## Rollback Process
1. Revert to previous tag
2. Redeploy
3. Notify team

## Common Issues
- "Database timeout": Check connection pool settings
- "File upload fails": Check Azure Blob permissions
```

---

## Key Differences: Team vs Solo

| Aspect | Solo (You) | Team |
|--------|-----------|------|
| **Decisions** | You decide everything | Discuss, document, get buy-in |
| **Code** | Write however you want | Follow team standards |
| **Changes** | Just change it | Create ticket, get approval |
| **Testing** | Optional | Required, reviewed |
| **Deployment** | Push whenever | Follow process, approvals |
| **Communication** | Talk to yourself | Slack, standups, PRs, docs |
| **Timeline** | Flexible | Fixed sprints, deadlines |

---

## What You Should Practice

To prepare for real team development:

### **1. Write Commits Like a Pro**
```bash
# Bad
git commit -m "fixed stuff"

# Good
git commit -m "feat: Add building CRUD endpoints

- Implement POST /api/buildings
- Add authorization checks
- Include integration tests
- Update Swagger documentation

Closes #123"
```

### **2. Document Decisions**
When you make a choice, write why:
```markdown
## Why SQLite for local dev?
- Quick setup
- No server required
- Matches SQL Server syntax
- Alternatives: In-memory (not persistent), Docker SQL Server (overkill)
```

### **3. Write Tests**
Teams expect tests:
```csharp
[Fact]
public async Task CreateBuilding_ValidInput_ReturnsCreated()
{
    // Arrange
    var dto = new CreateBuildingDto { Name = "HQ", ... };
    
    // Act
    var result = await _controller.CreateBuilding(1, dto);
    
    // Assert
    var createdResult = Assert.IsType<CreatedAtActionResult>(result);
    var building = Assert.IsType<BuildingDto>(createdResult.Value);
    Assert.Equal("HQ", building.Name);
}
```

### **4. Use Pull Requests (Even Solo)**
Practice on GitHub:
1. Create feature branch
2. Make changes
3. Push to GitHub
4. Create PR (even if you're the only reviewer)
5. Review your own code critically
6. Merge

---

## TL;DR - The Real Process

```
Week 0: Kickoff, requirements gathering, technical design
Week 1: Project setup, architecture decisions, sprint planning
Week 2-3: First sprint - MVP features
Week 4-5: Second sprint - core functionality
Week 6+: Iterate based on feedback

Throughout:
- Daily standups (15 min)
- Code reviews on every PR
- Testing expected
- Documentation updated
- Regular demos to stakeholders
```

**Not shown in tutorials:**
- The 10 Slack conversations to clarify requirements
- The 3 meetings to debate database choice
- The code review back-and-forth
- The "oh crap, we forgot about timezones" moment
- The last-minute design change
- The production bug that requires an emergency fix

---

Want me to help you start building EnergyTrack **the real way**? We can:

1. **Write the Technical Design Document** together
2. **Set up the project structure** (proper Clean Architecture)
3. **Create the first user story** and implement it with tests
4. **Practice PR workflow** (I'll "review" your code)
