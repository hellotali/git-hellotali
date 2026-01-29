# Applying Vertical Slice + Clean Architecture: Planning hellotali

**Author:** Harry Lo

**Date:** January 2026

---

## From Theory to Practice

In my previous article, [Clean Architecture vs Vertical Slice: Pragmatism Over Dogma](https://dev.to/harrykhlo/clean-architecture-vs-vertical-slice-pragmatism-over-dogma-in-modern-software-design-2co5), I explored how these two architectural patterns aren't opposites. They're complementary tools. The real skill is knowing how to balance them responsibly in real systems.

Now I'm putting that theory into practice with **hellotali**, a small hobby project I'm planning to build to support mental health and dementia care.

---

## Meaning of hellotali

- **"hello"** → A warm, friendly, non-clinical greeting
- **"Tali"** → A Hebrew name from *Tal* (טַל), meaning "dew," evoking freshness, renewal, and gentle support

Together, **hellotali** feels like "say hello to a caring companion that arrives as gently as morning dew."

---

## Project Status

**Early Stage** — This project is in active brainstorming and planning. Draft documentation is evolving and will be organized as the design solidifies.

---

## Why I'm Building This

Seeing how mental health and dementia challenges affect people around me has grown my compassion. I'm passionate about software development and want to help in a small, practical way, and I hope this can motivate others who care to take action too.

**What hellotali aims to do:**

- Provide simple daily mood check-ins (text or voice) with AI-powered analysis
- Show private trends to help people notice small changes over time
- Keep everything privacy-first and opt-in

**What it is NOT:**

- Not a diagnostic tool or replacement for professional care
- No crisis handling. Users should seek local emergency services when needed

This is a slow, hobby project alongside my full-time job as a software developer. No commitments expected from others, just learning and sharing as I go.

---

## Planned Stack (Under Evaluation)

| Layer | Technology | Notes |
|-------|------------|-------|
| **Frontend** | Angular 21, Material, PWA | Standalone components |
| **Backend** | .NET 10, EF Core, PostgreSQL | Vertical slice architecture |
| **AI** | Cloud or local options | Evaluating OpenAI, Azure, local fallback |

---

## Planned Architecture

Following the principles from my architecture article, hellotali plans to use **Vertical Slices for feature organization** with **Clean Architecture principles for engineering discipline**.

### Planned Project Structure

```
HelloTali.Features/          ← Vertical Slices (feature organization)
    CheckIn/                 ← Self-contained feature
    Trends/
    Billing/

HelloTali.Domain/            ← Clean Architecture (shared core)
    Entities/
    ValueObjects/

HelloTali.Contracts/         ← Dependency Inversion
    IAiAnalyzer
    IRepository

HelloTali.Infrastructure/    ← External dependencies
    AI Adapters
    EF Core
```

### AI Adapter Strategy (Planned)

One practical example of applying Open-Closed Principle:

```
Feature Handler
      ↓
  IAiAnalyzer (interface)
      ↓
┌─────────────┬─────────────┬─────────────┐
│  OpenAI     │   Azure     │   Local     │
│  Analyzer   │  Analyzer   │  Fallback   │
└─────────────┴─────────────┴─────────────┘
```

Adding a new AI provider means **adding a new class**, not modifying existing code. This is OCP in practice. The system is open for extension, closed for modification.

---

## Key Design Principles

1. **Start with Vertical Slices** — Organize by features, not technical layers
2. **Apply Clean Architecture where it adds value** — Contracts for DI, Domain for shared logic
3. **Keep slices independent** — Duplication across feature boundaries is healthy
4. **Use interfaces for flexibility** — Easy to swap AI providers or mock for testing

---

## Get Involved

This is an open hobby project. If you're interested:

- **Share feedback** on the idea and approach
- **Suggest resources** or guidelines I should follow
- **Connect me** with people interested in this space

**Project link:** [hellotali on GitHub](https://github.com/hellotali/git-hellotali)

---

## References

- [Clean Architecture vs Vertical Slice: Pragmatism Over Dogma](https://dev.to/harrykhlo/clean-architecture-vs-vertical-slice-pragmatism-over-dogma-in-modern-software-design-2co5)

---

*Harry Lo is a software developer exploring sustainable architecture and pragmatic system design. This project is part of his effort to apply engineering principles to meaningful, real-world problems.*
