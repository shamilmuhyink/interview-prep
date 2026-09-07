# EPAM Technical Managerial Round Questions

> **Focus Areas:** Project Architecture, Conflict Resolution, Crisis Management, Process Knowledge, Client-Readiness
> **Note:** The Managerial / Techno-Managerial round at EPAM assesses your ability to lead, handle production issues, and interact with clients. Questions are generally open-ended and require the STAR (Situation, Task, Action, Result) method.

---

### 1. 🏢 Explain the architecture of your current/most significant project. Why did you choose your specific tech stack?
**Answer:**
- **Preparation Tip:** Have a clear, high-level diagram in mind. 
- **Structure:**
  - **Overview:** What business problem does the project solve?
  - **Tech Stack Justification:** "We chose Spring Boot for rapid backend development and robust ecosystem. We used PostgreSQL for transactional consistency (ACID) and MongoDB for storing unstructured product catalogs."
  - **Microservices:** Explain how services communicate (REST vs gRPC vs Kafka).
  - **Infrastructure:** Mention Docker/Kubernetes for scalable deployment.
- **Key Focus:** Interviewers look for *why* you made decisions, trade-offs considered (e.g., consistency vs availability), and your understanding of the whole system, not just your specific module.

---

### 2. 🌐 Describe a difficult production problem you solved. Walk me through your debugging process.
**Answer:**
- **Use the STAR Method:**
  - **Situation:** "During the holiday season, our payment service experienced intermittent timeouts (0.5% failure rate)."
  - **Task:** "I needed to identify the bottleneck without bringing down the system."
  - **Action:** 
    1. Looked at centralized logs (ELK/Kibana) using the Trace ID. 
    2. Checked Grafana dashboards and noticed high DB connection pool usage.
    3. Identified that a third-party API was responding slowly, tying up our threads.
    4. Implemented a Circuit Breaker (Resilience4j) with a strict timeout and fallback mechanism.
  - **Result:** "Failures dropped to zero, and the system degraded gracefully instead of failing entirely."
- **Key Focus:** Demonstrates analytical skills, logical debugging, and knowledge of infrastructure monitoring tools.

---

### 3. 🏢 How do you prioritize tasks when business requirements change abruptly mid-sprint?
**Answer:**
- **Communication is Key:**
  1. Evaluate the impact of the new requirement on the current sprint goal.
  2. Discuss with the Product Owner / Scrum Master. If it's a P0 (critical blocker), swap it with an existing task of equivalent story points.
  3. Re-estimate and ensure the team agrees on the revised scope.
  4. Communicate the impact (delayed features) to stakeholders clearly.
- **Key Focus:** Shows Agile mindset, flexibility, and strong communication skills.

---

### 4. 🌐 Tell me about a time you disagreed with a senior stakeholder or architect on a technical decision. How did you resolve it?
**Answer:**
- **Example Scenario:** Disagreeing on using a Monolith vs Microservices for a new feature.
- **Action:** "I didn't argue based on opinions. Instead, I created a quick Proof of Concept (PoC) or gathered data on latency, deployment time, and cost."
- **Resolution:** "I presented the data objectively. We realized that while Microservices offered better scaling, the overhead for this specific small feature wasn't worth it, so we compromised on a modular monolith."
- **Key Focus:** Proves you handle conflict professionally, rely on data over ego, and can compromise.

---

### 5. 🏢 How do you handle code reviews in your team? What do you look for beyond syntax errors?
**Answer:**
- **Beyond Syntax:**
  1. **Business Logic & Edge Cases:** Does it actually solve the business requirement? What happens with nulls or boundary values?
  2. **Performance:** Are there N+1 query issues? Inefficient loops? 
  3. **Security:** Are there SQL injection risks or hardcoded secrets?
  4. **Maintainability:** Is the code readable, DRY (Don't Repeat Yourself), and SOLID? Are tests included?
- **Approach:** "I view code reviews as a mentoring opportunity, not a criticism session. I ask questions ('Have you considered what happens if X fails?') rather than giving direct commands."

---

### 6. 🌐 How do you manage and reduce technical debt in a fast-moving agile environment?
**Answer:**
- **Visibility:** Make tech debt visible by creating Jira tickets for it.
- **Allocation:** Advocate for allocating 15-20% of sprint capacity to addressing technical debt.
- **Prioritization:** Fix debt that directly impacts velocity (e.g., flaky tests slowing down CI/CD) or causes production incidents first.
- **Continuous Effort:** Follow the 'Boy Scout Rule' — always leave the codebase cleaner than you found it. 
- **Key Focus:** Shows maturity. Interviewers want to see that you view tech debt as a business problem (slowing down delivery) rather than just an aesthetic code problem.
