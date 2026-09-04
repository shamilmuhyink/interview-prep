# Module 11: Scenario-Based Questions (Bonus)

> **Scope:** Troubleshooting, Bottlenecks, Architecture Trade-offs, Scalability, and Concurrency
> **Questions:** 15 | **Critical:** 5 | **Coverage:** Product & Service-Based Companies | Sorted by interview frequency (descending)

---

## 🔴 CRITICAL / MUST-KNOW (Top 5)

### Q1. 🔴 [🌐] Your API suddenly takes 5 seconds instead of 200ms. How would you find the bottleneck?
**Answer:**
- **APM Tools:** Check Application Performance Monitoring (APM) tools like Datadog, New Relic, or AppDynamics to trace where the time is spent (DB, downstream API, or CPU).
- **Database Logs:** Look for slow queries. Ensure indexes are being utilized and check for table/row locks.
- **Thread Dumps:** Take thread dumps (using `jstack`) to see if threads are blocked waiting for external resources or locks.
- **GC Logs:** High garbage collection activity (Stop-The-World pauses) can drastically increase response times.
- **Infrastructure:** Check for CPU throttling, network latency, or high memory usage on the pod/server.

### Q2. 🔴 [🌐] Two users update the same record at the same time. How do you handle it?
**Answer:**
- **Optimistic Locking:** Best for high-read, low-write scenarios. Add a `@Version` field to the entity. Hibernate checks the version during update; if it differs, it throws an `OptimisticLockException`.
- **Pessimistic Locking:** Best for high-write contention. Lock the row at the database level (`SELECT ... FOR UPDATE`) so the second user waits until the first transaction completes.

| Strategy | Contention | Performance | Database Lock |
|----------|------------|-------------|---------------|
| Optimistic | Low | High | None (Application level) |
| Pessimistic| High | Lower | Yes (DB level) |

### Q3. 🔴 [🌐] A payment request reaches your API twice. How do you prevent duplicate payments?
**Answer:**
- **Idempotency Keys:** Require clients to send a unique `Idempotency-Key` header with the request.
- **Database Constraint:** Store the key in a database table with a `UNIQUE` constraint. If the second request tries to insert the same key, the database throws a constraint violation.
- **Distributed Cache/Lock:** Use Redis `SETNX` (Set if Not eXists) to acquire a lock using the transaction ID before processing.

### Q4. 🔴 [🌐] A downstream service takes 10 seconds to respond. What would you do?
**Answer:**
- **Timeouts:** Configure aggressive read and connect timeouts (e.g., 2 seconds) on your HTTP client so your threads are not blocked indefinitely.
- **Circuit Breaker:** Wrap the call using a tool like Resilience4j. If failures or timeouts exceed a threshold, open the circuit to fail fast and prevent cascading failure.
- **Fallback Mechanism:** Return a cached response, a default value, or an error message (Graceful Degradation).
- **Asynchronous Processing:** If the response isn't needed immediately, send a message to a queue and let a background worker handle the downstream call.

### Q5. 🔴 [🌐] Your DB connection pool is exhausted. What would you check first?
**Answer:**
- **Connection Leaks:** Ensure connections are properly closed. In Java, this means ensuring connections are returned to the pool (using try-with-resources or framework-level transaction management).
- **Long-Running Transactions:** A query taking too long holds the connection. Check DB slow query logs.
- **Traffic Spikes:** If traffic legitimately increased, the pool size (e.g., HikariCP `maximum-pool-size`) might simply be too small for the load.
- **Deadlocks:** Check if threads are deadlocked holding DB connections while waiting for other resources.

---

## 🟡 HIGH FREQUENCY (Questions 6–12)

### Q6. 🟡 [🌐] Your application throws `OutOfMemoryError` only in production. What would you investigate?
**Answer:**
- **Heap Dump Analysis:** Configure `-XX:+HeapDumpOnOutOfMemoryError` to automatically generate a heap dump. Use tools like Eclipse MAT (Memory Analyzer Tool) to find the largest objects and GC roots.
- **Memory Leaks:** Look for static collections (Lists, Maps) that continually grow without bounds, or unclosed resources (Streams, Connections).
- **Traffic/Load:** Production might handle payloads larger than QA. Ensure pagination is used for DB queries rather than loading entire tables into memory.

### Q7. 🟡 [🌐] Kafka consumer lag keeps increasing. How would you troubleshoot it?
**Answer:**
- **Processing Time:** The consumer is taking too long to process messages. Optimize the processing logic or batch database inserts.
- **Concurrency:** Increase the number of partitions for the topic and deploy more consumer instances (up to the number of partitions) to parallelize processing.
- **Throughput Settings:** Tweak consumer configurations like `max.poll.records` to process smaller batches faster, preventing session timeouts.
- **Errors/Retries:** If the consumer is endlessly retrying a failing message (poison pill), it will block the partition. Implement a Dead Letter Queue (DLQ).

### Q8. 🟡 [🌐] One microservice goes down during an order transaction. How should your system behave?
**Answer:**
- **Saga Pattern:** Do not use distributed transactions (2PC). Instead, use a Saga (Choreography or Orchestration).
- **Compensating Transactions:** If the order service succeeds but the payment service is down, the system should eventually execute a compensating transaction to cancel the order.
- **Eventual Consistency:** Use a message broker to queue the request. Once the microservice recovers, it consumes the message and completes its part of the transaction.

### Q9. 🟡 [🌐] Your retry mechanism starts increasing traffic instead of reducing failures. Why?
**Answer:**
- **Retry Storms:** When a downstream service is struggling, aggressive retries multiply the load, knocking it down completely.
- **Exponential Backoff:** Retries must have exponential backoff (e.g., wait 1s, then 2s, then 4s).
- **Jitter:** Add randomness (jitter) to the backoff so all retrying clients don't hit the downstream service at the exact same millisecond.
- **Circuit Breaker Integration:** A circuit breaker should stop retries entirely if the service is confirmed down.

### Q10. 🟡 [🌐] A scheduled job executes twice after deployment. How would you prevent duplicate execution?
**Answer:**
- **Distributed Locking:** Use a tool like **ShedLock** or **Redisson** (Redis) to ensure only one instance of the application acquires the lock to execute the job.
- **Database Flag:** Create a job execution table. Use an atomic `UPDATE` or `INSERT` with a unique constraint for the specific job timestamp.
- **Clustered Schedulers:** Use frameworks designed for distributed environments, such as Quartz with clustered mode enabled.

### Q11. 🟡 [🌐] You need to introduce a new field in an API without breaking existing clients. How?
**Answer:**
- **Additive Changes Only:** Add the field as optional in the response JSON. Existing clients using strict deserialization might break if they don't ignore unknown fields (ensure `@JsonIgnoreProperties(ignoreUnknown = true)` is standard client practice).
- **API Versioning:** If the change fundamentally alters behavior, introduce a new endpoint (e.g., `/v2/resource`) while keeping `/v1/resource` active.
- **Default Values:** If it's a new required field in a request payload, provide a sensible default at the controller level so old clients can still communicate successfully.

### Q12. 🟡 [🌐] Your logs don't let you trace a request across multiple microservices. How would you improve this?
**Answer:**
- **Distributed Tracing:** Implement **Trace ID** and **Span ID** propagation using tools like Micrometer Tracing (formerly Spring Cloud Sleuth) and OpenTelemetry.
- **MDC (Mapped Diagnostic Context):** Ensure the Trace ID is injected into the SLF4J MDC so every log statement automatically includes it.
- **Log Aggregation:** Export logs to a centralized system (ELK Stack, Splunk, Datadog) where you can query by the unique Trace ID to see the entire lifecycle across all services.

---

## 🟢 GOOD TO KNOW (Questions 13–15)

### Q13. 🟢 [🌐] The DB query takes 50ms, but the API takes 3 seconds. Where could the remaining time go?
**Answer:**
- **N+1 Query Problem:** The initial query was 50ms, but ORM (Hibernate) lazy-loaded child entities inside a loop, triggering hundreds of additional queries.
- **Network Latency:** High latency between the application server and the database, or the client and the application.
- **Serialization:** Converting massive Java objects into JSON (Jackson) can block the CPU for seconds.
- **Connection Acquisition:** The query is fast, but waiting for a connection from an exhausted pool took 2.9 seconds.

### Q14. 🟢 [🌐] Your application handles 1,000 requests/sec today. Tomorrow it needs to handle 10,000. What changes would you consider?
**Answer:**
- **Horizontal Scaling:** Deploy more instances of the application behind a Load Balancer (Kubernetes HPA).
- **Caching:** Introduce Redis or Memcached for read-heavy endpoints to offload the database.
- **Database Scaling:** Implement read replicas for read queries, and consider sharding/partitioning if writes become a bottleneck.
- **Asynchronous Processing:** Move synchronous heavy tasks (like sending emails or generating PDFs) to a background worker queue (Kafka/RabbitMQ).

### Q15. 🟢 [🌐] Your service is receiving duplicate requests during a network timeout. How would you make the operation safe?
**Answer:**
- **Idempotency:** Design the API to be idempotent (e.g., `PUT` or `DELETE`). Applying the operation multiple times should have the same effect as applying it once.
- **Transactional Outbox Pattern:** If the operation involves updating the database and publishing an event, write both to the database in a single transaction. A separate relay publisher reads the outbox table, ensuring the event is published exactly once or at least once with idempotency on the consumer side.
