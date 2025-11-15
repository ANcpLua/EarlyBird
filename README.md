# EarlyBird - Software Architecture Exercises

> [!IMPORTANT]
> **This repository has been archived.** All exercises are now consolidated in the [Software-Architecture](https://github.com/ANcpLua/Software-Architecture) repository for easier navigation.
>
> **Find EarlyBird exercises at:**
> - Evening 2: [ISearchProduct Interface](https://github.com/ANcpLua/Software-Architecture/tree/main/evening-2/01-isearchproduct-specification)
> - Evening 2: [Interface Quality Review](https://github.com/ANcpLua/Software-Architecture/tree/main/evening-2/02-interface-quality-review)
> - Evening 2: [IList Interface Design](https://github.com/ANcpLua/Software-Architecture/tree/main/evening-2/03-ilist-interface-design)
> - Evening 3: [A Bigger Application Core](https://github.com/ANcpLua/Software-Architecture/tree/main/evening-3/04-bigger-application-core)

Software architecture exercises for a breakfast delivery system, demonstrating O-Interface design, quality assurance, and application core architecture principles.

Part of the Software Architecture course exercise collection.

## Table of Contents

- [Project Status](#project-status)
- [Exercises](#exercises)
- [Technology Stack](#technology-stack)
- [Repository Structure](#repository-structure)
- [Related Repositories](#related-repositories)
- [Course Context](#course-context)

## Project Status

**Active** - Complete set of interface design and application core architecture exercises.

> [!NOTE]
> This repository contains exercises ArchitecturalQuality03-07 and EarlyBird12 from Evening 2 and Evening 3 of the Software Architecture course. All exercises use the EarlyBird breakfast delivery case study.

## Exercises

### Exercise 1: ISearchProduct Interface Specification
**Exercise ID:** ArchitecturalQuality03 | **Evening:** 2 | **Type:** Group

Complete interface specification with thread-safety, null-safety, and comprehensive documentation.

**Files:**
- [Specification](01-isearchproduct-specification/isearchproduct-interface.md)
- [Exercise Slide 140](01-isearchproduct-specification/exercise-slide-140.pdf)
- [Requirements](01-isearchproduct-specification/earlybird-requirements-v150.pdf)

---

### Exercise 2: Interface Quality Review
**Exercise IDs:** ArchitecturalQuality04 (checklist) + ArchitecturalQuality05 (review) | **Evening:** 2 | **Type:** Group

Extended 11-question quality checklist and peer review of example interface (scored 41/100).

**Files:**
- [Quality Checklist](02-interface-quality-review/interface-quality-checklist.md)
- [Sample Peer Review](02-interface-quality-review/peer-review-isearchproduct.md)
- [Exercise Slide 145](02-interface-quality-review/exercise-slide-145.pdf)
- [Requirements](02-interface-quality-review/earlybird-requirements-v150.pdf)

---

### Exercise 3: IList Interface Design & Review
**Exercise IDs:** ArchitecturalQuality05 (design) + ArchitecturalQuality07 (review) | **Evening:** 2 | **Type:** Group

Generic list interface design with Interface Segregation Principle analysis (scored 45/100).

**Files:**
- [Peer Review](03-ilist-interface-design/peer-review-ilist.md)
- [Exercise Slides 147-148](03-ilist-interface-design/exercise-slide-147-148.pdf)
- [Requirements](03-ilist-interface-design/earlybird-requirements-v150.pdf)

> [!CAUTION]
> The IList exercise demonstrates a critical covariance error in the original specification. See peer review for detailed analysis of the Interface Segregation Principle violation.

---

### Exercise 4: Application Core Architecture
**Exercise ID:** EarlyBird12 | **Evening:** 3 | **Type:** Home

Application core design separating stable business logic from volatile technology concerns, with change impact analysis for three evolution scenarios.

**Files:**
- [Architecture Design](04-application-core-architecture/application-core-design.md)
- [Exercise Slide 228](04-application-core-architecture/exercise-slide-228.pdf)
- [Requirements](04-application-core-architecture/earlybird-requirements-v150.pdf)

> [!TIP]
> EarlyBird12 should be completed after Mars02 (Mars Moon Calculator). The small application core principles apply directly to larger systems.

---

## Technology Stack

- **Framework:** .NET 10.0
- **Language:** C# (with ImplicitUsings and Nullable enabled)
- **Output Type:** Console Application
- **Documentation:** GitHub-flavored Markdown (CommonMark spec)

## Repository Structure

```
EarlyBird/
├── CLAUDE.md                               # In-memory specification
├── EarlyBird.sln                           # .NET solution
└── EarlyBird/
    ├── README.md                           # This file
    ├── EarlyBird.csproj                    # .NET 10 console app
    ├── 01-isearchproduct-specification/    # ArchitecturalQuality03
    │   ├── isearchproduct-interface.md
    │   ├── exercise-slide-140.pdf
    │   └── earlybird-requirements-v150.pdf
    ├── 02-interface-quality-review/        # ArchitecturalQuality04+05
    │   ├── interface-quality-checklist.md
    │   ├── peer-review-isearchproduct.md
    │   ├── exercise-slide-145.pdf
    │   └── earlybird-requirements-v150.pdf
    ├── 03-ilist-interface-design/          # ArchitecturalQuality05+07
    │   ├── peer-review-ilist.md
    │   ├── exercise-slide-147-148.pdf
    │   └── earlybird-requirements-v150.pdf
    └── 04-application-core-architecture/   # EarlyBird12
        ├── application-core-design.md
        ├── exercise-slide-228.pdf
        └── earlybird-requirements-v150.pdf
```

## Related Repositories

This repository is part of a collection of Software Architecture course exercises:

| Repository | Purpose |
|------------|---------|
| [earlybird-sdd](https://github.com/ANcpLua/earlybird-sdd) | Hub repository with documentation and learning path |
| **EarlyBird** (this repo) | Main application exercises (interface design, application core) |
| [EarlyBirdAI](https://github.com/ANcpLua/EarlyBirdAI) | AI components for EarlyBird |
| [MateMate](https://github.com/ANcpLua/MateMate) | Mermaid diagram tooling |
| [Optional-Home-Exercise-Tools-Corner](https://github.com/ANcpLua/Optional-Home-Exercise-Tools-Corner) | Complete exercise list and tools |
| [Mars](https://github.com/ANcpLua/Mars) | Architecture exercises and Mars moon calculator |

## Course Context

**Software Architecture** - Blended Learning Course

- **ECTS**: 2-3 ECTS (50-75 hours total workload)
- **Format**: 4 evenings (2+4+4+4 hours each)
- **Focus**: Clean architecture, design decisions, and software engineering best practices

### Course Philosophy

> "Good engineers write code. Great engineers make good decisions."
>
> — Raul Junco (@RaulJuncoV)

**Key Principles:**
1. Build what you need, not what you imagine
2. Logs, monitoring, and error handling aren't optional
3. Test your code before your users do
4. The right tool > the latest tool

### EarlyBird in the Course

The EarlyBird breakfast delivery system serves as the primary case study for:
- **Evening 2**: Interface design and quality (ArchitecturalQuality03-07)
- **Evening 3**: Application core architecture (EarlyBird12)
- **Evening 3**: AI-assisted architecture (ArchitectureDevelopment02)

### Additional Resources

- [Learning Path](https://github.com/ANcpLua/earlybird-sdd/blob/main/docs/LEARNING_PATH.md) - Structured course progression
- [Exercise List](https://github.com/ANcpLua/Optional-Home-Exercise-Tools-Corner/blob/main/README.md) - All available exercises
- [EarlyBird Documentation](https://github.com/ANcpLua/earlybird-sdd/tree/main/docs) - Project documentation hub

---

**Course Completion:** All 21 exercises
**Total Time Investment:** ~40 hours (4 evenings × 4 hours + ~24 hours homework)
**Prerequisite Knowledge:** Software development experience, basic OOP
