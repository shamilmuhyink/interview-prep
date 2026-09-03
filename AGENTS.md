# 🤖 AI Agent Instructions — Interview Preparation Repository

> **Purpose**: These are directives for any AI agent working on this repository. Read and follow these rules **before** creating, editing, or reorganizing any file in this project.

---

## 0. Agent Role

**Act as a mentor for software engineering professionals.** When working on this repository:

1. **Curate & Fetch** — Fetch interview questions from platforms like AmbitionBox, Glassdoor, Reddit, PrepInsta, GeeksforGeeks, LinkedIn, and Quora etc. Filter them for quality, accuracy, and relevance to the target role (Senior Full Stack Java Developer, 5+ years).
2. **Classify** — Determine which module (Section 2) each question belongs to and place it in the correct file.
3. **Rank** — Assign a frequency tier (🔴 → 🟡 → 🟢) based on the cross-company frequency table (Section 5). Place the question at the correct position within the file.
4. **Sync to Module-Wise** — Whenever a question is added to a `Company_wise/` file, check if it already exists in the corresponding `Module_wise/` file. **If it does not exist there, add it.** Module-wise files are the canonical reference — every question must have a home there.
5. **Re-rank on every addition** — After adding a new question, re-evaluate the frequency tier of surrounding questions. If a topic now appears in more companies, promote it to a higher tier. Update the cross-company frequency table (Section 5) if needed.

---

## 1. Repository Structure

This repository has **4 top-level directories**. Understand what each contains before touching anything.

```
Java-interview-prep/
├── Module_wise/          # Questions grouped by technical module (10 files)
├── Company_wise/         # Questions grouped by company (5 subdirectories)
├── Master/               # Consolidated master Q&A and PDF guide
├── DSA/                  # Standalone DSA problem sets (LeetCode, NeetCode)
└── INTERVIEW_PREP_GUIDE.md   ← YOU ARE HERE (this file)
```

---

## 2. Module Taxonomy

All questions in this repository belong to one of **10 modules**. When adding, moving, or tagging a question, you **must** classify it into exactly one of these modules.

| # | Module ID | Module Name | Key Topics | File |
|---|-----------|-------------|-----------|------|
| 1 | `CORE_JAVA` | Language (Core Java) | Multithreading, JVM Internals, Collections, Functional Programming | `Module_wise/Module_1_language_core_java.md` |
| 2 | `SPRING_BOOT` | Backend Framework (Spring Boot) | IoC/DI, AOP, Security, REST, Spring MVC | `Module_wise/Module_2_backend_framework_springboot.md` |
| 3 | `DBMS` | DBMS (MySQL, PostgreSQL) | SQL Optimization, Indexing, Transactions, ACID | `Module_wise/Module_3_dbms_mysql_postgresql.md` |
| 4 | `ORM` | ORM (JPA, Hibernate) | Caching, N+1 Problem, Entity Lifecycle, Relationships | `Module_wise/Module_4_ORM_jpa_hibernate.md` |
| 5 | `ANGULAR` | Frontend Framework (Angular) | Lifecycle, State Management, Rendering, Backend Integration | `Module_wise/Module_5_frontend_framework_angular.md` |
| 6 | `MICROSERVICES` | Microservices | Service Discovery, Distributed Tracing, Resilience, API Gateway | `Module_wise/Module_6_Microservices.md` |
| 7 | `DESIGN_PATTERNS` | Design Patterns | Creational, Structural, Behavioral Patterns, Anti-patterns | `Module_wise/Module_7_Design_patterns.md` |
| 8 | `DEVOPS` | DevOps (Docker, K8s, AWS) | Containerization, Orchestration, CI/CD, Deployment | `Module_wise/Module_8_DevOps.md` |
| 9 | `DSA` | Data Structures & Algorithms | Core Algorithms, Complexity Analysis, Custom DS Design | `Module_wise/Module_9_DSA.md` |
| 10 | `SYSTEM_DESIGN` | System Design & Architecture (Bonus) | SOLID, Scalability, CQRS, Event Sourcing, CAP Theorem | `Module_wise/Bonus_System_Design.md` |

---

## 3. Company-Wise Files

There are **7 companies** tracked. Each has its own subdirectory under `Company_wise/`.

| Company | Directory | Files | Notes |
|---------|-----------|-------|-------|
| **TCS** | `Company_wise/TCS/` | `TCS_Interview_QA.md` (150 Qs) | Largest dataset. Covers all modules. Service-based focus. |
| **Deloitte** | `Company_wise/deloitte/` | `technical_round.md` (50 Qs), `managerial_round.md` (15 Qs), `hr_round.md` (10 Qs) | Split by round type. |
| **Experion** | `Company_wise/Experion/` | `experion_interview.md` (50 Qs) | Q&A format (Interviewer/Candidate). Includes a JD PDF. |
| **IBS** | `Company_wise/IBS/` | `IBS_round_1_dsa.md` (30 Qs), `IBS_round_2_managerial.md` (66 Qs) | Split by round. R1 is pure coding/DSA. |
| **Tekion** | `Company_wise/Tekion/` | `tekion_java_staff_backend_questions.md` | Staff-level. Heavy on System Design and concurrency. |
| **Synechron** | `Company_wise/Synechron/` | `technical_round.md` (20 Qs), `managerial_round.md` (4 Qs), `hr_round.md` (4 Qs) | Focuses on deep Java, Concurrency, and Microservices Resiliency. Split by round type. |
| **Infosys** | `Company_wise/Infosys/` | `infosys_interview_questions.md` (50 Qs) | Service-based focus. Full Stack Java topics. |

---

## 4. Frequency Ranking Rules

> **CRITICAL**: Questions in `Module_wise/` files are already sorted by descending interview frequency. Questions in `Company_wise/` files are sorted by frequency within that company. **Do not re-sort by topic or alphabetically.**

### 4.1 Module-Wise Priority Tiers (Preserve These)

Each module file uses a **3-tier priority system**. When adding a new question, place it in the correct tier based on how many companies ask it.

| Tier | Label | Position | Selection Criteria |
|------|-------|----------|-------------------|
| 🔴 Critical | `CRITICAL / MUST-KNOW` | Q1–Q5 | Asked by 4+ companies, or universally expected for the role |
| 🟡 High | `HIGH FREQUENCY` | Q6–Q12 | Asked by 2–3 companies, strong differentiator |
| 🟢 Good | `GOOD TO KNOW` | Q13–Q20+ | Asked by 1 company or shows depth, impressive when nailed |

### 4.2 Company-Wise Ranking (Re-rank by Frequency, NOT Topic)

When editing or adding questions to any `Company_wise/` file:
- **DO** order questions by how frequently the company asks them (most frequent first).
- **CRITICAL INSTRUCTION:** Company-wise, don't list question-answers topic-wise. Instead, rank them completely based on frequency. DO NOT group questions by topic/module within company files. The ordering must reflect the company's actual interview patterns.
- If frequency data is unavailable, place new questions **at the end** of the file.

### 4.3 Cross-Company Frequency Tiers (For Master Files)

When updating `Master/Interview_QA_Master.md` or creating consolidated views, rank topics by how many of the 7 companies ask them:

| Tier | Symbol | Companies Asking | Action |
|------|--------|-----------------|--------|
| Universal | 🔴⚡ | 7/7 companies | Must appear in top section of any master file |
| Very High | 🟠 | 5-6/7 companies | Must appear in second section |
| High | 🟡 | 3-4/7 companies | Third section |
| Medium | 🟢 | 2/7 companies | Fourth section |
| Specific | 🔵 | 1/7 companies | Bottom section or company-specific only |

---

## 5. Cross-Company Frequency Reference

Use this table as the **source of truth** when deciding question priority, ordering, or which questions to add/remove. Topics are listed in descending frequency order.

### 5.1 Universal (7/7 Companies) — Never Remove These

| Topic | Module | TCS | Deloitte | Experion | IBS | Tekion | Synechron | Infosys |
|-------|--------|-----|----------|---------|-----|--------|-----------|---------|
| HashMap internals (put/get, treeification) | `CORE_JAVA` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| HashMap vs ConcurrentHashMap (thread safety) | `CORE_JAVA` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| OOP Concepts (4 pillars with examples) | `CORE_JAVA` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Java 8 Streams API (filter, map, collect) | `CORE_JAVA` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Microservices vs Monolithic architecture | `MICROSERVICES` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| SOLID Principles | `SYSTEM_DESIGN` | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

### 5.2 Very High (4/5 Companies)

| Topic | Module | Companies |
|-------|--------|-----------|
| String vs StringBuilder vs StringBuffer | `CORE_JAVA` | TCS, Deloitte, Experion, IBS |
| Garbage Collection (G1, ZGC, tuning) | `CORE_JAVA` | TCS, Deloitte, Experion, Tekion |
| Spring DI/IoC + Constructor Injection | `SPRING_BOOT` | TCS, Deloitte, Experion, IBS |
| Spring Boot Auto-configuration internals | `SPRING_BOOT` | TCS, Deloitte, IBS, Tekion |
| ACID properties + Transaction Isolation | `DBMS` | TCS, Deloitte, Experion, IBS |
| SQL Joins (INNER, LEFT, RIGHT, FULL) | `DBMS` | TCS, Deloitte, Experion, IBS |
| JWT/OAuth2 auth flow in Spring Security | `SPRING_BOOT` | TCS, Deloitte, Experion, IBS |
| Circuit Breaker (Resilience4j) | `MICROSERVICES` | TCS, Deloitte, IBS, Tekion |
| Docker (Image vs Container, Dockerfile) | `DEVOPS` | TCS, Deloitte, Experion, IBS |
| CI/CD pipeline concepts | `DEVOPS` | TCS, Deloitte, Experion, IBS |
| Runnable vs Callable | `CORE_JAVA` | TCS, Deloitte, Experion, IBS |
| Exception Handling (Checked vs Unchecked) | `CORE_JAVA` | TCS, Deloitte, Experion, IBS |
| Two Sum problem | `DSA` | TCS, Deloitte, Experion, IBS |
| Singleton Pattern (thread-safe) | `DESIGN_PATTERNS` | TCS, Deloitte, Experion, IBS |
| API Gateway pattern | `MICROSERVICES` | TCS, Deloitte, Experion, IBS |
| AWS Services (S3, EC2, RDS) | `DEVOPS` | TCS, Deloitte, Experion, IBS |
| Palindrome check (String) | `DSA` | TCS, Deloitte, Experion, IBS |
| Character/Word frequency counting | `DSA` | TCS, Deloitte, Experion, IBS |
| Factory + Builder patterns | `DESIGN_PATTERNS` | TCS, Deloitte, Experion, IBS |
| Database Indexing (B-Tree, covering) | `DBMS` | TCS, Deloitte, IBS, Tekion |
| Global Exception Handling (`@ControllerAdvice`) | `SPRING_BOOT` | TCS, Deloitte, Experion, IBS |

### 5.3 High (3/5 Companies)

| Topic | Module | Companies |
|-------|--------|-----------|
| `@Transactional` propagation & isolation | `ORM` | TCS, Deloitte, IBS |
| `map()` vs `flatMap()` in Streams | `CORE_JAVA` | TCS, Deloitte, IBS |
| `volatile` vs `synchronized` | `CORE_JAVA` | TCS, IBS, Tekion |
| ArrayList vs LinkedList | `CORE_JAVA` | TCS, Experion, IBS |
| Spring Bean Lifecycle | `SPRING_BOOT` | TCS, Deloitte, IBS |
| Service Discovery (Eureka, K8s DNS) | `MICROSERVICES` | TCS, IBS, Tekion |
| Saga Pattern (distributed transactions) | `MICROSERVICES` | TCS, IBS, Tekion |
| N+1 Query Problem (Hibernate) | `ORM` | TCS, Deloitte, Experion |
| Reverse a Linked List | `DSA` | TCS, Deloitte, Experion |
| Detect cycle in Linked List (Floyd's) | `DSA` | TCS, Deloitte, IBS |
| Kubernetes basics (Pod, Service) | `DEVOPS` | TCS, Deloitte, Experion |
| Angular Lifecycle Hooks | `ANGULAR` | TCS, Deloitte, IBS |
| Comparable vs Comparator | `CORE_JAVA` | TCS, IBS, Deloitte |
| Normalization (1NF–3NF) | `DBMS` | Deloitte, Experion, IBS |
| GROUP BY / HAVING | `DBMS` | TCS, Deloitte, IBS |
| `@RestController` vs `@Controller` | `SPRING_BOOT` | TCS, Deloitte, IBS |
| Functional Interfaces + Lambda | `CORE_JAVA` | TCS, Deloitte, IBS |
| Spring Boot Actuator | `SPRING_BOOT` | TCS, Deloitte, Experion |
| Angular Routing + Lazy Loading | `ANGULAR` | Deloitte, Experion, IBS |
| Observable vs Promise (RxJS) | `ANGULAR` | Experion, IBS, Deloitte |
| `@Component` vs `@Service` vs `@Repository` | `SPRING_BOOT` | TCS, Deloitte, IBS |
| Agile Scrum methodology | Process | TCS, Deloitte, Experion |

### 5.4 Medium (2/5 Companies)

| Topic | Companies |
|-------|-----------|
| CompletableFuture patterns | TCS, Tekion |
| Virtual Threads (Java 21) | Experion, Tekion |
| FetchType.LAZY vs EAGER | TCS, Deloitte |
| Optimistic vs Pessimistic Locking | Experion, Tekion |
| Spring WebFlux vs MVC | TCS, Tekion |
| CQRS / Event Sourcing | TCS, IBS |
| SQL vs NoSQL selection | IBS, Tekion |
| Kafka Architecture (topics, partitions) | TCS, Tekion |
| Redis Caching strategies | Deloitte, Tekion |
| `wait()` vs `sleep()` | TCS, IBS |
| Second highest element in array | Experion, IBS |
| Anagram check/grouping | Deloitte, IBS |
| git merge vs git rebase | Experion, IBS |
| Window Functions (ROW_NUMBER, RANK) | TCS, Deloitte |
| Query Optimization (EXPLAIN) | Deloitte, Tekion |
| Docker Compose | Experion, IBS |
| Spring Profiles | Experion, IBS |
| Angular Interceptors | Experion, IBS |
| `throw` vs `throws` | TCS, IBS |
| Serialization / `transient` | TCS, IBS |
| CAP Theorem | Experion, IBS |
| `final`, `finally`, `finalize` | TCS, Deloitte |
| Abstract Class vs Interface | TCS, Deloitte |

---

## 6. Rules for Modifying Files

### 6.1 Adding a New Question

1. **Classify** it into exactly one module from Section 2.
2. **Check** the cross-company frequency table (Section 5) to determine which tier it belongs to.
3. **Place** the question in the correct tier position within the module file (🔴 → 🟡 → 🟢).
4. **Tag** the question with the company coverage icon: 🌐 Both | 🏢 Product | 🏬 Service.
5. **Follow the answer format**:
   ```
   1. 🎯 One-sentence definitive summary
   2. 📖 Structured explanation (concept → mechanics → trade-offs)
   3. 💻 Production-quality code snippet (where applicable)
   4. ⚠️ Edge cases & common pitfalls
   ```
6. If adding to a `Company_wise/` file, maintain the **frequency-based order** (most asked first), not topic grouping.

### 6.2 Removing a Question

- **NEVER** remove a question from Tier 1 (Universal, 5/5 companies) without explicit user approval.
- Before removing any question, check if it appears in other files (module-wise AND company-wise). Update cross-references.
- Removing from a company file does NOT remove it from the module file, and vice versa.

### 6.3 Editing an Existing Question

- **Preserve** the existing priority tier and position unless the frequency data has changed.
- **Preserve** all existing code snippets, tables, and `⚠️ Pitfall` sections.
- **Do not** change the question numbering scheme (Q1, Q2, ...) within a file — other files may reference these numbers.
- When updating code examples, target **Java 21+, Spring Boot 3.3+, Angular 18+**.

### 6.4 Reorganizing or Restructuring

- The `Module_wise/` directory must maintain exactly **10 files** (9 modules + 1 bonus).
- Each module file header must include: Scope, Question count, Critical count, Coverage tags, and sort order note.
- Company files must remain in their respective subdirectory. Do not merge company files across companies.
- The `Master/Interview_QA_Master.md` must remain a **flat, topic-grouped** master document.

---

## 7. File Format Conventions

### 7.1 Module Files (`Module_wise/*.md`)

```markdown
# Module N: [Module Name]

> **Scope:** [Key Topics]
> **Questions:** [Count] | **Critical:** [Count] | **Coverage:** Product & Service-Based Companies | Sorted by interview frequency (descending)

---

## 🔴 CRITICAL / MUST-KNOW (Top 5)

### Q1. 🔴 [🌐|🏢|🏬] [Question text]
[Answer following the 4-part format]

---

## 🟡 HIGH FREQUENCY (Questions 6–12)

### Q6. [🌐|🏢|🏬] [Question text]
...

## 🟢 GOOD TO KNOW (Questions 13–20)

### Q13. [🌐|🏢|🏬] [Question text]
...
```

### 7.2 Company Files (`Company_wise/**/*.md`)

- Maintain the company's existing format (some use `### Q1:`, others use `### 1.`, others use `**Q1. Interviewer:**`).
- **Do not unify** question numbering format across companies — each has its own style.
- Questions must be ordered by **descending frequency within that company**, not by module/topic.

### 7.3 Company Coverage Tags

| Tag | When to Use |
|-----|-------------|
| 🌐 **Both** | Asked at both product and service companies |
| 🏢 **Product** | Primarily product companies (Google, Amazon, Flipkart, Atlassian, Razorpay, Tekion) |
| 🏬 **Service** | Primarily service companies (TCS, Infosys, Wipro, HCL, Cognizant, Deloitte) |

---

## 8. Tech Stack Constraints

When writing or updating code examples, use these versions:

| Technology | Version | Notes |
|-----------|---------|-------|
| Java | 21+ | Use records, sealed classes, virtual threads where relevant |
| Spring Boot | 3.3+ | `SecurityFilterChain` (not `WebSecurityConfigurerAdapter`) |
| Angular | 18+ | Standalone components, signals where applicable |
| PostgreSQL | 16+ | Use modern SQL features (CTEs, window functions) |
| Docker | Latest | Multi-stage builds preferred |
| Kubernetes | 1.28+ | — |

---

## 9. Quality Checklist Before Any Commit

Before finalizing changes to any file in this repository, verify:

- [ ] Question is classified into exactly one module (Section 2)
- [ ] Question is placed in the correct frequency tier (Section 4)
- [ ] Cross-company frequency table (Section 5) is still accurate after the change
- [ ] Answer follows the 4-part format (summary → explanation → code → pitfalls)
- [ ] Code examples compile with the tech stack versions in Section 8
- [ ] Existing comments, pitfalls, and cross-references are preserved
- [ ] Company files maintain frequency-based ordering (not topic-based)
- [ ] Module files maintain the 3-tier structure (🔴 → 🟡 → 🟢)

---

> *This instruction file governs all AI-assisted modifications to the `Java-interview-prep` repository.*
> *Last updated: September 2026 | 5 company datasets | 10 modules | 400+ questions tracked*
