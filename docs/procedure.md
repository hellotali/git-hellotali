## Development Procedures (hellotali)

### 0. Prerequisites
- Windows 10/11, PowerShell
- .NET 10 SDK (`dotnet --version` should show 10.x)
- Node.js 20+ and npm
- Angular CLI (`npm i -g @angular/cli@21`) optional
- Docker Desktop (for local PostgreSQL)
- OpenAI API key (for MVP analysis)

### 0b. Create `C:/Repos` with correct permissions (Windows)
If you prefer storing repos at `C:/Repos` (outside your user profile), create it once with proper ACLs. Use an elevated PowerShell (Run as Administrator):
```
New-Item -Path C:\Repos -ItemType Directory -Force | Out-Null
$u = $env:USERNAME
icacls C:\Repos /grant "$u:(OI)(CI)F"
```
Notes:
- Admin rights are required to create at the drive root. Granting `(OI)(CI)F` gives your user Full control on this folder and all children.
- Alternative without elevation: use `%USERPROFILE%\Repos` (no special permissions needed).

### 0a. Create the GitHub repository (first time)
1) Sign in to your GitHub account `hellotali` and create a repo:
   - Go to `https://github.com/new` (ensure owner is `hellotali`).
   - Repository name: `git-hellotali` (recommendation).
   - Visibility: your choice (private/public).
   - Do NOT initialize with README/.gitignore (we have files locally).
   - Click Create repository. You should now have `https://github.com/hellotali/git-hellotali`.

2) Optional: set up SSH (recommended) or use HTTPS
   - SSH (generate key):
     ```
     ssh-keygen -t ed25519 -C "your_email@example.com"
     # press Enter for default path; add a passphrase if desired
     Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub
     ```
     Copy the printed key and add it to GitHub → Settings → SSH and GPG keys → New SSH key.
   - HTTPS requires no key setup but will prompt for credentials/tokens on push.

3) Initialize local repo and push main branch
```
cd C:\Users\harry\MyWorks\git-hellotali
git init
git branch -M main
git add .
git commit -m "chore: initial documentation and infra setup"
# Choose ONE of the following remotes:
# SSH:
git remote add origin git@github.com:hellotali/git-hellotali.git
# HTTPS:
# git remote add origin https://github.com/hellotali/git-hellotali.git
git push -u origin main
```

4) Create environments (development, staging, production)
   - GitHub → Repo → Settings → Environments → New environment
   - Add `development`, `staging`, `production`.
   - You can later add environment secrets (e.g., `OPENAI_API_KEY`, DB connection) used by CI/CD.

Reference: `https://github.com/hellotali` profile exists with no public repos yet, so this will be your first repo.

### 1. Clone and Repo Layout
```
git clone <repo-url>
cd git-hellotali
```
Structure: `backend/`, `frontend/`, `docs/`, `infra/`.

### 1b. Initialize local repository and push to a new empty remote
If your local folder already has files but the remote repo is newly created and empty, run:

1) Initialize and make first commit
```
cd C:\Users\harry\MyWorks\git-hellotali
git init
git add .
git commit -m "chore: initial commit"
git branch -M main
```

2) Set identity for this repo (or rely on includeIf mapping)
```
git config user.name "HelloTali Harry"
git config user.email "hellotaliharry@gmail.com"  # or your GitHub noreply email
```

3) Add remote (choose SSH alias or HTTPS)
```
# SSH with host alias (recommended; see section 14)
git remote add origin git@github-hellotali:hellotali/git-hellotali.git
# OR HTTPS
# git remote add origin https://github.com/hellotali/git-hellotali.git
```

4) First push
```
git push -u origin main
```

5) Optional: auto-apply identity for this folder via includeIf (global)
Add to `%USERPROFILE%/.gitconfig`:
```
[includeIf "gitdir:C:/Users/harry/MyWorks/git-hellotali/"]
    path = ~/.gitconfig-hellotali
```
Then create `~/.gitconfig-hellotali`:
```
[user]
    name = HelloTali Harry
    email = hellotaliharry@gmail.com
```

6) Create environment branches (develop, staging) and push
```
# from main
git checkout -b develop
git push -u origin develop

git checkout main
git checkout -b staging
git push -u origin staging

git checkout main
```

7) Branch protection and environment mapping (GitHub UI)
- In GitHub → Repo → Settings → Branches → Add branch protection rule:
  - `main`: require PR, status checks, and manual approval for production deploys
  - `staging`: require PR and checks; auto-deploy to staging
- CI/CD mapping guideline:
  - `develop` → deploy to development environment on push
  - `staging` → deploy to staging environment on push
  - `main` (or a release tag) → deploy to production with manual approval

### 2a. Scaffold the codebase (Vertical Slice Architecture)

This creates the solution/projects with vertical slice architecture using .NET 10 and Angular 21.

1) Backend solution and projects (Vertical Slice Structure)
```powershell
# Create backend folder structure
mkdir -Force backend/src
cd backend

# Create solution
dotnet new sln -n HelloTali

# Create projects following vertical slice architecture
dotnet new classlib -n HelloTali.Domain -o src/HelloTali.Domain
dotnet new classlib -n HelloTali.Contracts -o src/HelloTali.Contracts
dotnet new classlib -n HelloTali.Features -o src/HelloTali.Features
dotnet new classlib -n HelloTali.Infrastructure -o src/HelloTali.Infrastructure
dotnet new web -n HelloTali.Api -o src/HelloTali.Api

# Add projects to solution
dotnet sln add src/HelloTali.Domain/HelloTali.Domain.csproj
dotnet sln add src/HelloTali.Contracts/HelloTali.Contracts.csproj
dotnet sln add src/HelloTali.Features/HelloTali.Features.csproj
dotnet sln add src/HelloTali.Infrastructure/HelloTali.Infrastructure.csproj
dotnet sln add src/HelloTali.Api/HelloTali.Api.csproj

# Add project references (Vertical Slice dependencies)
# Domain has no dependencies (pure business logic)
# Contracts depends on Domain
dotnet add src/HelloTali.Contracts/HelloTali.Contracts.csproj reference src/HelloTali.Domain/HelloTali.Domain.csproj
# Features depends on Domain and Contracts
dotnet add src/HelloTali.Features/HelloTali.Features.csproj reference src/HelloTali.Domain/HelloTali.Domain.csproj
dotnet add src/HelloTali.Features/HelloTali.Features.csproj reference src/HelloTali.Contracts/HelloTali.Contracts.csproj
# Infrastructure implements Contracts
dotnet add src/HelloTali.Infrastructure/HelloTali.Infrastructure.csproj reference src/HelloTali.Domain/HelloTali.Domain.csproj
dotnet add src/HelloTali.Infrastructure/HelloTali.Infrastructure.csproj reference src/HelloTali.Contracts/HelloTali.Contracts.csproj
# Api references everything
dotnet add src/HelloTali.Api/HelloTali.Api.csproj reference src/HelloTali.Features/HelloTali.Features.csproj
dotnet add src/HelloTali.Api/HelloTali.Api.csproj reference src/HelloTali.Infrastructure/HelloTali.Infrastructure.csproj
```

2) Backend packages (.NET 10, EF Core PostgreSQL, OpenAPI)
```powershell
# Infrastructure packages
dotnet add src/HelloTali.Infrastructure/HelloTali.Infrastructure.csproj package Npgsql.EntityFrameworkCore.PostgreSQL
dotnet add src/HelloTali.Infrastructure/HelloTali.Infrastructure.csproj package Microsoft.EntityFrameworkCore.Design

# API packages
dotnet add src/HelloTali.Api/HelloTali.Api.csproj package Swashbuckle.AspNetCore

# Install EF Core tools globally
dotnet tool install --global dotnet-ef
```

3) Create folder structure for vertical slices
```powershell
# Features folder structure
mkdir -Force src/HelloTali.Features/CheckIn/CreateCheckIn
mkdir -Force src/HelloTali.Features/CheckIn/GetCheckIn
mkdir -Force src/HelloTali.Features/CheckIn/Models
mkdir -Force src/HelloTali.Features/Trends/GetTrends
mkdir -Force src/HelloTali.Features/Trends/Models
mkdir -Force src/HelloTali.Features/Billing/GetUsage
mkdir -Force src/HelloTali.Features/Billing/Models

# Domain folder structure
mkdir -Force src/HelloTali.Domain/Entities
mkdir -Force src/HelloTali.Domain/ValueObjects
mkdir -Force src/HelloTali.Domain/Services

# Contracts folder structure
mkdir -Force src/HelloTali.Contracts/AI
mkdir -Force src/HelloTali.Contracts/Persistence
mkdir -Force src/HelloTali.Contracts/Services

# Infrastructure folder structure
mkdir -Force src/HelloTali.Infrastructure/Persistence/Repositories
mkdir -Force src/HelloTali.Infrastructure/AI
mkdir -Force src/HelloTali.Infrastructure/Services
```

4) Database and migrations
```powershell
cd src/HelloTali.Infrastructure
dotnet ef migrations add InitialCreate --startup-project ../HelloTali.Api
cd ../HelloTali.Api
dotnet ef database update
```

5) Frontend app scaffold (Angular 21)
```powershell
cd ../../..
mkdir -Force frontend
cd frontend
npx -y @angular/cli@21 new hellotali-app --standalone --routing --style=scss --ssr=false --skip-git --strict --defaults
cd hellotali-app
npx -y @angular/cli@21 add @angular/material --skip-confirmation --theme=indigo-pink --typography --animations
npx -y @angular/cli@21 add @angular/pwa --skip-confirmation --project hellotali-app
```

6) Create frontend feature structure
```powershell
# Features (vertical slices)
npx -y ng g c features/check-in/pages/check-in-page --standalone
npx -y ng g c features/check-in/components/question-card --standalone
npx -y ng g s features/check-in/services/check-in --flat
npx -y ng g c features/trends/pages/trends-page --standalone
npx -y ng g c features/trends/components/trend-chart --standalone
npx -y ng g s features/trends/services/trends --flat
npx -y ng g c features/billing/pages/usage-page --standalone
npx -y ng g s features/billing/services/billing --flat

# Core services
npx -y ng g s core/services/api --flat
npx -y ng g s core/services/storage --flat

# Shared components
npx -y ng g c shared/components/loading-spinner --standalone
npx -y ng g c shared/components/navbar --standalone
```

7) Verify builds
```powershell
# Backend (in backend/src/HelloTali.Api)
dotnet build && dotnet run

# Frontend (in frontend/hellotali-app)
npm run build && npm start
```

### 2b. Backend code skeletons (paste into files)

Create these minimal files to get the API compiling. Namespaces assume project names as created.

1) `backend/HelloTali.Domain/Entities/CheckIn.cs`
```csharp
namespace HelloTali.Domain.Entities;

public class CheckIn
{
    public Guid Id { get; set; }
    public string UserId { get; set; } = string.Empty;
    public DateTime SubmittedAt { get; set; } = DateTime.UtcNow;
    public string Channel { get; set; } = "text"; // text|voice
    public string? TextAnswer { get; set; }
    public string? AudioUrl { get; set; }
    public AnalysisResult? Analysis { get; set; }
}
```

2) `backend/HelloTali.Domain/Entities/AnalysisResult.cs`
```csharp
namespace HelloTali.Domain.Entities;

public class AnalysisResult
{
    public Guid Id { get; set; }
    public Guid CheckInId { get; set; }
    public double SentimentScore { get; set; }
    public string EmotionLabel { get; set; } = "neutral";
    public string CognitiveSignalsJson { get; set; } = "{}";
    public string RiskLevel { get; set; } = "none";
    public DateTime AnalyzedAt { get; set; } = DateTime.UtcNow;
}
```

3) `backend/HelloTali.Application/AI/IAiAnalyzer.cs`
```csharp
using HelloTali.Domain.Entities;

namespace HelloTali.Application.AI;

public interface IAiAnalyzer
{
    Task<AnalysisResult> AnalyzeTextAsync(string userId, string text, CancellationToken ct = default);
}
```

4) `backend/HelloTali.Infrastructure/Data/HelloTaliDbContext.cs`
```csharp
using HelloTali.Domain.Entities;
using Microsoft.EntityFrameworkCore;

namespace HelloTali.Infrastructure.Data;

public class HelloTaliDbContext : DbContext
{
    public HelloTaliDbContext(DbContextOptions<HelloTaliDbContext> options) : base(options) { }

    public DbSet<CheckIn> CheckIns => Set<CheckIn>();
    public DbSet<AnalysisResult> AnalysisResults => Set<AnalysisResult>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<CheckIn>()
            .HasOne(c => c.Analysis)
            .WithOne()
            .HasForeignKey<AnalysisResult>(a => a.CheckInId);
    }
}
```

5) `backend/HelloTali.Infrastructure/AI/OpenAiGptAnalyzer.cs`
```csharp
using System.Net.Http.Json;
using HelloTali.Application.AI;
using HelloTali.Domain.Entities;

namespace HelloTali.Infrastructure.AI;

public class OpenAiGptAnalyzer : IAiAnalyzer
{
    private readonly HttpClient _http;
    private readonly string _model;

    public OpenAiGptAnalyzer(HttpClient http)
    {
        _http = http;
        _model = Environment.GetEnvironmentVariable("OPENAI_MODEL") ?? "gpt-4o-mini";
        var key = Environment.GetEnvironmentVariable("OPENAI_API_KEY");
        if (!string.IsNullOrWhiteSpace(key))
        {
            _http.DefaultRequestHeaders.Authorization = new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", key);
        }
        _http.BaseAddress = new Uri("https://api.openai.com/v1/");
    }

    public async Task<AnalysisResult> AnalyzeTextAsync(string userId, string text, CancellationToken ct = default)
    {
        // Minimal placeholder: you can replace with official SDK later
        var prompt = $"Classify sentiment (-1..1) and label for: {text}";
        var req = new
        {
            model = _model,
            messages = new[] { new { role = "user", content = prompt } },
            temperature = 0.0
        };
        using var response = await _http.PostAsJsonAsync("chat/completions", req, ct);
        response.EnsureSuccessStatusCode();
        var now = DateTime.UtcNow;
        return new AnalysisResult
        {
            Id = Guid.NewGuid(),
            SentimentScore = 0, // TODO: parse response
            EmotionLabel = "neutral",
            CognitiveSignalsJson = "{}",
            RiskLevel = "none",
            AnalyzedAt = now
        };
    }
}
```

6) `backend/HelloTali.Api/Program.cs` (key registrations)
```csharp
using HelloTali.Application.AI;
using HelloTali.Infrastructure.AI;
using HelloTali.Infrastructure.Data;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var cs = Environment.GetEnvironmentVariable("DATABASE_URL") ?? "Host=localhost;Port=5432;Database=hellotali;Username=dev;Password=devpass";
builder.Services.AddDbContext<HelloTaliDbContext>(opt => opt.UseNpgsql(cs));

builder.Services.AddHttpClient<IAiAnalyzer, OpenAiGptAnalyzer>();

builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(policy => policy
        .WithOrigins("http://localhost:4200")
        .AllowAnyHeader()
        .AllowAnyMethod());
});

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseCors();
app.MapControllers();

app.Run();
```

7) `backend/HelloTali.Api/Controllers/CheckInsController.cs`
```csharp
using HelloTali.Application.AI;
using HelloTali.Domain.Entities;
using HelloTali.Infrastructure.Data;
using Microsoft.AspNetCore.Mvc;

namespace HelloTali.Api.Controllers;

[ApiController]
[Route("checkins")]
public class CheckInsController : ControllerBase
{
    private readonly HelloTaliDbContext _db;
    private readonly IAiAnalyzer _ai;

    public CheckInsController(HelloTaliDbContext db, IAiAnalyzer ai)
    {
        _db = db; _ai = ai;
    }

    public record CheckInRequest(string UserId, string Channel, string? TextAnswer);

    [HttpPost]
    public async Task<IActionResult> Create([FromBody] CheckInRequest req, CancellationToken ct)
    {
        var checkIn = new CheckIn
        {
            Id = Guid.NewGuid(),
            UserId = req.UserId,
            Channel = req.Channel,
            TextAnswer = req.TextAnswer,
            SubmittedAt = DateTime.UtcNow
        };
        _db.CheckIns.Add(checkIn);
        await _db.SaveChangesAsync(ct);

        var analysis = await _ai.AnalyzeTextAsync(req.UserId, req.TextAnswer ?? string.Empty, ct);
        analysis.Id = Guid.NewGuid();
        analysis.CheckInId = checkIn.Id;
        _db.AnalysisResults.Add(analysis);
        await _db.SaveChangesAsync(ct);

        return Ok(new { checkInId = checkIn.Id, analysis.SentimentScore, analysis.EmotionLabel, analysis.RiskLevel });
    }
}
```

8) `backend/HelloTali.Api/Controllers/TrendsController.cs`
```csharp
using HelloTali.Infrastructure.Data;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace HelloTali.Api.Controllers;

[ApiController]
[Route("trends")]
public class TrendsController : ControllerBase
{
    private readonly HelloTaliDbContext _db;
    public TrendsController(HelloTaliDbContext db) { _db = db; }

    [HttpGet]
    public async Task<IActionResult> Get([FromQuery] string userId, [FromQuery] string range = "30d")
    {
        var since = DateTime.UtcNow.AddDays(-30);
        var daily = await _db.AnalysisResults
            .Where(a => _db.CheckIns.Any(c => c.Id == a.CheckInId && c.UserId == userId && c.SubmittedAt >= since))
            .OrderBy(a => a.AnalyzedAt)
            .Select(a => new { date = a.AnalyzedAt.Date, score = a.SentimentScore })
            .ToListAsync();
        return Ok(new { daily });
    }
}
```

### 2. Local Infrastructure (PostgreSQL)
Create `infra/docker-compose.postgres.yml` (see app plan) and run:
```
docker compose -f infra/docker-compose.postgres.yml up -d
```
Default connection:
```
Host=localhost;Port=5432;Database=hellotali;Username=dev;Password=devpass
```

### 3. Backend – Run (.NET 8)
Set environment variables (PowerShell example):
```
$env:DATABASE_URL = "Host=localhost;Port=5432;Database=hellotali;Username=dev;Password=devpass"
$env:AI_PROVIDER = "openai"
$env:OPENAI_API_KEY = "<your_key>"
```
Start API:
```
cd backend/HelloTali.Api
dotnet restore
dotnet ef database update   # if migrations exist
dotnet run
```
Swagger: `https://localhost:5001/swagger`

### 4. Frontend – Run (Angular 18 PWA)
```
cd frontend/hellotali-app
npm install
npm start
```
Dev server: `http://localhost:4200`. Configure `API_BASE_URL` in `src/environments/environment.ts` if needed.

### 5. Testing
- Backend: `dotnet test` in solution root.
- Frontend: `npm test` (unit), `npm run e2e` if configured.

### 6. Branching and Environments
- Branches: `develop` → `staging` → `main` (production).
- Deploy targets: development, staging, production.
- Use versioned releases via tags; conventional commits for changelog.

### 7. Migrations (EF Core)
- Add migration:
```
cd backend/HelloTali.Infrastructure
dotnet ef migrations add <Name>
```
- Apply migration (local):
```
cd ../HelloTali.Api
dotnet ef database update
```

### 8. OpenAI Configuration
- Required envs:
  - `AI_PROVIDER=openai`
  - `OPENAI_API_KEY=<key>`
  - Optional: `OPENAI_MODEL=gpt-4o-mini` (or your chosen model)

### 9. CI/CD (overview)
- Backend pipeline: build, test, PostgreSQL service for integration tests, build Docker image, push (staging), manual approve (prod).
- Frontend pipeline: install, lint, test, build, deploy to static host/CDN.

### 10. Code Quality
- Linters and formatters enabled; commit hooks optional (Husky for frontend).
- PRs require at least one review; unit tests for new logic; docs updated.

### 11. Safety and Content
- Display consent and disclaimer in app.
- Crisis keyword detection returns helpline info; no automated escalation.

### 12. Troubleshooting
- DB connection refused: ensure Docker is running and port 5432 is free.
- OpenAI errors: verify key and organization limits; backoff on 429.
- CORS: ensure API allows `http://localhost:4200` in development.

### 13. Viewing the Architecture Diagram
- The diagram and Mermaid source are in `docs/architecture.md`.
- GitHub renders Mermaid automatically in Markdown; just view the file in the repo.
- Local preview: open in VS Code and use Markdown Preview. If Mermaid doesn’t render, install a Mermaid preview extension.
- Exporting an image (optional):
  - Put the Mermaid block into `docs/architecture.mmd` and run:
    - `npm i -g @mermaid-js/mermaid-cli`
    - `mmdc -i docs/architecture.mmd -o docs/architecture.png`

### 14. Multiple GitHub accounts on one machine (Windows, SSH per-folder)

Goal: Use four GitHub accounts with separate SSH keys, per-folder identities, and correct access.

Accounts
- `harrykhlo` — email: `harrykhlo@gmail.com`
- `brickwareharry` — email: `brickware.harry@gmail.com`
- `hopmates` — email: `harryhopmates@gmail.com`
- `hellotali` — email: `hellotaliharry@gmail.com`

1) Create folder layout (one root per account)
```
mkdir C:\Repos\harrykhlo, C:\Repos\brickwareharry, C:\Repos\hopmates, C:\Repos\hellotali
```

2) Generate SSH keys (one per account) \
[(Note: In PowerShell, you should quote that -f path (otherwise \.ssh looks like a property access on the string variable)]

```
ssh-keygen -t ed25519 -C "harrykhlo@gmail.com" -f $env:USERPROFILE\.ssh\id_ed25519_harrykhlo
ssh-keygen -t ed25519 -C "brickware.harry@gmail.com" -f $env:USERPROFILE\.ssh\id_ed25519_brickwareharry
ssh-keygen -t ed25519 -C "harryhopmates@gmail.com" -f $env:USERPROFILE\.ssh\id_ed25519_hopmates
ssh-keygen -t ed25519 -C "hellotaliharry@gmail.com" -f $env:USERPROFILE\.ssh\id_ed25519_hellotali
```

3) Start ssh-agent and add keys (persist across sessions)
```
# (Run PowerShell as Administrator for the first line)
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
ssh-add $env:USERPROFILE\.ssh\id_ed25519_harrykhlo
ssh-add $env:USERPROFILE\.ssh\id_ed25519_brickwareharry
ssh-add $env:USERPROFILE\.ssh\id_ed25519_hopmates
ssh-add $env:USERPROFILE\.ssh\id_ed25519_hellotali
```

4) Upload public keys to each GitHub account
```
Get-Content $env:USERPROFILE\.ssh\id_ed25519_harrykhlo.pub
Get-Content $env:USERPROFILE\.ssh\id_ed25519_brickwareharry.pub
Get-Content $env:USERPROFILE\.ssh\id_ed25519_hopmates.pub
Get-Content $env:USERPROFILE\.ssh\id_ed25519_hellotali.pub
```
Copy each key to: GitHub → Settings → SSH and GPG keys → New SSH key.

5) Configure SSH host aliases (`~/.ssh/config`)
Create (or edit) `%USERPROFILE%\.ssh\config` with:
```
Host github-harrykhlo
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_harrykhlo
  IdentitiesOnly yes

Host github-brickwareharry
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_brickwareharry
  IdentitiesOnly yes

Host github-hopmates
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_hopmates
  IdentitiesOnly yes

Host github-hellotali
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_hellotali
  IdentitiesOnly yes
```

6) Configure per-folder git identity (conditional includes)
Edit global `%USERPROFILE%\.gitconfig` and append:
```
[includeIf "gitdir:C:/Repos/harrykhlo/"]
    path = ~/.gitconfig-harrykhlo
[includeIf "gitdir:C:/Repos/brickwareharry/"]
    path = ~/.gitconfig-brickwareharry
[includeIf "gitdir:C:/Repos/hopmates/"]
    path = ~/.gitconfig-hopmates
[includeIf "gitdir:C:/Repos/hellotali/"]
    path = ~/.gitconfig-hellotali
```

Create these files with identities:
```
echo [user]\n\tname = Harry Khlo\n\temail = harrykhlo@gmail.com > $env:USERPROFILE\.gitconfig-harrykhlo
echo [user]\n\tname = Brickware Harry\n\temail = brickware.harry@gmail.com > $env:USERPROFILE\.gitconfig-brickwareharry
echo [user]\n\tname = Hopmates Harry\n\temail = harryhopmates@gmail.com > $env:USERPROFILE\.gitconfig-hopmates
echo [user]\n\tname = HelloTali Harry\n\temail = hellotaliharry@gmail.com > $env:USERPROFILE\.gitconfig-hellotali
```

7) Clone or set remotes with the correct alias
- Example (clone under HelloTali):
```
git clone git@github-hellotali:hellotali/git-hellotali.git C:\Repos\hellotali\git-hellotali
```
- Or set an existing repo’s remote:
```
cd C:\Repos\hellotali\git-hellotali
git remote set-url origin git@github-hellotali:hellotali/git-hellotali.git
```

8) Verify each account mapping
```
ssh -T git@github-harrykhlo
ssh -T git@github-brickwareharry
ssh -T git@github-hopmates
ssh -T git@github-hellotali
```
You should see the greeting indicating which account you’re authenticated as.

9) Optional per-repo enforcement
In any repo, force a specific key regardless of folder:
```
git config core.sshCommand "ssh -i ~/.ssh/id_ed25519_hellotali -o IdentitiesOnly=yes"
```

10) HTTPS alternative (if not using SSH)
- Use Git Credential Manager (GCM) on Windows; each repo uses a different remote URL and cached token.
- Still set `user.name` and `user.email` via conditional includes above.

Troubleshooting
- If the wrong account is used, check: remote URL alias, ssh-agent loaded keys, and `~/.ssh/config` order.
- For `includeIf`, ensure Windows paths use forward slashes and end with a trailing slash.

#### 14b. Use multiple accounts without reorganizing folders (per-repo setup)
If your repos live together (e.g., `C:/Users/harry/MyWorks/*`) and are mixed across accounts, keep them as-is. Configure each repo individually:

1) Ensure SSH keys and host aliases exist (see section 14 steps 2–5)

2) For each repo, set the correct identity and, optionally, force the SSH key
```
cd <path-to-your-repo>
git config user.name "HelloTali Harry"
git config user.email "hellotaliharry@gmail.com"
# Optional hard bind to the right SSH key (not needed if your remote uses a host alias)
git config core.sshCommand "ssh -i ~/.ssh/id_ed25519_hellotali -o IdentitiesOnly=yes"
```
Repeat with the appropriate name/email for other accounts:
- `harrykhlo@gmail.com`
- `brickware.harry@gmail.com`
- `harryhopmates@gmail.com`

3) Point the remote to the matching host alias (so the right key is chosen)
```
git remote -v
git remote set-url origin git@github-hellotali:hellotali/<repo>.git         # for HelloTali
# or
git remote set-url origin git@github-harrykhlo:harrykhlo/<repo>.git         # for harrykhlo
git remote set-url origin git@github-brickwareharry:brickwareharry/<repo>.git
git remote set-url origin git@github-hopmates:hopmates/<repo>.git
```

4) Verify
```
git remote -v
ssh -T git@github-hellotali      # or the alias you used for this repo
git config user.name
git config user.email
```

5) Optional: per-repo identity via includeIf (explicit repo path)
Instead of storing `user.*` in the repo config, you can map specific repo paths to an account config in your global `~/.gitconfig`:
```
[includeIf "gitdir:C:/Users/harry/MyWorks/git-hellotali/"]
    path = ~/.gitconfig-hellotali
```
Add one entry per repo you want to map (adjust the path for each repo). This avoids reorganizing folders and still applies the right identity automatically.

### 15. Moving the project folder without losing setup or chat

If you want to move this project from the current path to `C:/Repos/hellotali/git-hellotali` and keep everything working, follow these steps.

1) Save work and commit
```
git add .
git commit -m "chore: save before moving project folder"
```

2) Close your IDE/editor (Cursor/VS Code)

3) Create destination folders and move
```
mkdir C:\Repos\hellotali
robocopy "C:\Users\harry\MyWorks\git-hellotali" "C:\Repos\hellotali\git-hellotali" /MIR
```
Verify the new folder has the files, then optionally remove the old folder.

4) Update Git per-folder identity includeIf path
Edit `%USERPROFILE%/.gitconfig` and change the HelloTali include to the new path:
```
[includeIf "gitdir:C:/Repos/hellotali/"]
    path = ~/.gitconfig-hellotali
```

5) (Optional) Keep a junction at the old path (helps any old shortcuts)
```
mklink /J "C:\Users\harry\MyWorks\git-hellotali" "C:\Repos\hellotali\git-hellotali"
```

6) Reopen the project in your IDE from the NEW path
- In Cursor/VS Code: File → Open Folder → `C:/Repos/hellotali/git-hellotali`.
- Chat history: moving folders does not delete your chat. If needed, open Chat History or reopen the previous session from the app. As a backup, you can copy key parts of this chat into `docs/chat-notes.md` before moving.

7) Verify Git and remotes still work
```
git remote -v
git fetch
ssh -T git@github-hellotali
```

8) Update any other includeIf rules for other accounts if you move their folders too

15a) Move to `C:/Repos/hellotali` (no nested subfolder)

If you prefer the project root to be `C:/Repos/hellotali` (not `.../git-hellotali`), use:

```
robocopy "C:\Users\harry\MyWorks\git-hellotali" "C:\Repos\hellotali" /MIR
```

Then create the junction from the old path to the new root (run elevated PowerShell):

```
rmdir "C:\Users\harry\MyWorks\git-hellotali" /S /Q
mklink /J "C:\Users\harry\MyWorks\git-hellotali" "C:\Repos\hellotali"
```

Update your `includeIf` rule accordingly:

```
[includeIf "gitdir:C:/Repos/hellotali/"]
    path = ~/.gitconfig-hellotali
```

Reopen the folder `C:/Repos/hellotali` in your IDE.

Windows user control considerations
- Using `C:/Repos/hellotali` is fine. Ensure you created `C:/Repos` with permissions for your user (see section 0b). Otherwise, you may hit write-permission issues (e.g., node_modules install, EF migrations).
- Some tools may prompt for elevation if `C:/Repos` lacks write ACLs for your user. Fix by running the `icacls` command in 0b.

