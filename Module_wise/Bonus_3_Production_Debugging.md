# Module 12: Production Debugging & Observability (Bonus 3)

> **Scope:** Debugging, Troubleshooting, Observability, Performance Tuning, and Microservices Resilience
> **Questions:** 25 | **Critical:** 5 | **Coverage:** Product & Service-Based Companies | Sorted by interview frequency (descending)

---

## 🔴 CRITICAL / MUST-KNOW (Top 5)

### Q1. 🔴 [🌐] API is slow in production — where do you start debugging?
**Answer:**
- **Confirm Scope:** Identify which endpoint, service, users, and time period are affected.
- **Check Metrics:** Check API latency, error rate, CPU, memory, thread pool, and database metrics.
- **Distributed Tracing & Logs:** Use tracing to see whether the service is waiting on a downstream dependency (DB, Kafka, or another API).
- **Baseline Comparison:** Compare current latency with the normal baseline before making changes.

📊 **Debugging Strategy**
| Step | Action | Tool |
|------|--------|------|
| 1 | Identify Symptoms | Datadog, Prometheus |
| 2 | Isolate Bottleneck | OpenTelemetry, Splunk |
| 3 | Fix Safely | Config change, Rollback |

### Q2. 🔴 [🌐] High CPU usage after deployment — what will you check?
**Answer:**
- **Check Service/Instance:** Confirm which instance has high CPU and if it started immediately after the release.
- **Application Metrics:** Check thread activity and recent code/config changes.
- **Code Issues:** Look for infinite/large loops, excessive object processing, high request traffic, or expensive DB operations.
- **Thread Dumps/Profiling:** Take a thread dump or use profiling tools to find CPU-consuming code.
- **Rollback:** If the release clearly caused the issue, consider a rollback while investigating.

### Q3. 🔴 [🌐] Memory usage keeps increasing — possible causes?
**Answer:**
- **Retained Objects:** Large collections or objects retained longer than necessary.
- **Unbounded Structures:** Unbounded caches, static references, or `ThreadLocal` misuse.
- **Bulk Loading:** Loading too many DB records at once instead of pagination/streaming.
- **Large Payloads:** Large responses, files, or buffers held in memory.
- **Real Memory Leak:** An actual memory leak or an undersized heap under increased traffic.

### Q4. 🔴 [🌐] What if DB queries suddenly become slow?
**Answer:**
- **Check DB Metrics:** Look at CPU, connections, locks, wait time, and slow-query logs.
- **Execution Plan:** Identify the exact query and compare its execution plan with normal behavior using `EXPLAIN`.
- **Database Changes:** Check indexes, data growth, joins, filters, and recent schema/query changes.
- **Concurrency:** Check whether the application is creating too many concurrent queries.
- **Verification:** Fix the query and verify latency under realistic traffic.

### Q5. 🔴 [🌐] What happens if the thread pool is exhausted?
**Answer:**
- **Request Queuing/Rejection:** New requests may wait in the queue, become slow, or be rejected depending on the executor configuration.
- **Symptoms:** High latency, request timeouts, and rejected tasks.
- **Analysis:** Check active threads, queue size, and task execution time. Find why tasks are slow (e.g., DB, external calls, locks, or CPU).
- **Resolution:** Use appropriate pool sizing, timeouts, and backpressure rather than blindly increasing threads.

---

## 🟡 HIGH FREQUENCY (Questions 6–12)

### Q6. 🟡 [🌐] How do you identify memory leaks in Java?
**Answer:**
- **Monitor Heap & GC:** A leak often shows old-generation usage growing after repeated GC cycles.
- **Heap Dumps:** Take heap dumps at suitable points and compare retained objects.
- **Suspects:** Look for unexpectedly large collections, static maps, caches, listeners, or `ThreadLocal` values.
- **Tools:** Use Eclipse MAT, VisualVM, or Java Flight Recorder to identify dominators and retained memory.

### Q7. 🟡 [🌐] How do you analyze heap dumps?
**Answer:**
- **Open in Tool:** Open the heap dump in a tool such as Eclipse MAT.
- **Check Memory Distribution:** Check total heap, biggest object types, and retained heap.
- **Dominator Tree:** Look at the Dominator Tree to find objects retaining large amounts of memory.
- **Trace References:** Trace references to understand why those objects are still reachable.
- **Fix Root Cause:** Fix the code rather than simply increasing heap size.

### Q8. 🟡 [🌐] How do you analyze a thread dump?
**Answer:**
- **Thread States:** Look for `BLOCKED`, `WAITING`, and `RUNNABLE` threads.
- **Contention:** Check whether many threads are waiting on the same lock, DB connection, or external service.
- **Deadlocks:** Look for deadlocks and repeated stack traces.
- **Multiple Dumps:** Compare multiple thread dumps a few seconds apart to see whether the same threads remain stuck.
- **Correlate:** Relate the dump to thread-pool, CPU, and request metrics.

### Q9. 🟡 [🌐] How do you debug connection pool exhaustion?
**Answer:**
- **Pool Metrics:** Check active, idle, and maximum connections; also check wait time/timeouts.
- **Application Logs:** Check application logs for connection acquisition timeout errors.
- **Long-Running Transactions:** Look for long-running DB queries or transactions holding connections.
- **Connection Leaks:** Verify that connections are always closed/released.
- **Capacity:** Check whether traffic increased or the pool size/configuration is too small.

### Q10. 🟡 [🌐] How do you handle high latency in APIs?
**Answer:**
- **Breakdown Latency:** Break total latency into application processing, DB time, and downstream service time.
- **Tracing:** Use tracing to identify the slowest segment.
- **Optimize Bottleneck:** Optimize the actual bottleneck: query, network call, serialization, code, or dependency.
- **Caching/Async:** Use caching or asynchronous processing where appropriate.
- **Timeouts:** Set sensible timeouts so one dependency cannot hold requests indefinitely.

### Q11. 🟡 [🌐] What if one microservice is slowing the entire system?
**Answer:**
- **Identify:** Use metrics and distributed tracing to identify the dependency.
- **Check Health:** Check its CPU, memory, thread pool, DB, and downstream dependencies.
- **Protect Callers:** Protect callers with timeouts, circuit breakers, and controlled retries.
- **Optimize:** Scale or optimize the unhealthy service and reduce unnecessary synchronous calls.
- **Avoid Storms:** Verify that recovery of one service does not create a traffic/retry storm.

### Q12. 🟡 [🌐] How do you handle cascading failures?
**Answer:**
- **Resource Isolation:** Prevent one failing service from consuming all resources of its callers.
- **Resilience Patterns:** Use timeouts, circuit breakers, bulkheads, and bounded retries.
- **Async Communication:** Prefer asynchronous communication for non-critical work.
- **Graceful Degradation:** Return fallback data or degrade features gracefully where possible.
- **Monitoring:** Monitor dependency health and recovery carefully.

---

## 🟢 GOOD TO KNOW (Questions 13–25)

### Q13. 🟢 [🌐] How do you debug timeout issues?
**Answer:**
- **Identify Timeout Type:** Identify which timeout occurred: client, API Gateway, HTTP client, DB, or message-processing timeout.
- **Trace Request:** Use logs and trace IDs to follow one request across services.
- **Downstream Health:** Check whether the downstream service was slow, unavailable, or unreachable.
- **Review Configurations:** Review timeout values and connection/read timeouts.
- **Fix Delay:** Do not solve every timeout by simply increasing the timeout; fix the underlying delay.

### Q14. 🟢 [🌐] What is a circuit breaker real use case?
**Answer:**
- **Purpose:** A circuit breaker temporarily stops calls to a failing/slow dependency after failures cross a threshold.
- **Prevent Cascading Failure:** It prevents repeated calls from consuming application threads.
- **Recovery:** After a recovery period, it allows limited test calls; if healthy, it closes again.
- **Use Case:** E-commerce Order Service calls Notification Service. If notification is failing repeatedly, the circuit opens and order processing continues without blocking.

### Q15. 🟢 [🌐] How do you debug Kafka consumer lag?
**Answer:**
- **Check Lag:** Check consumer lag by topic/partition and identify affected consumer groups.
- **Processing Time:** Check consumer processing time, consumer count, partition distribution, and rebalance activity.
- **Downstream Latency:** Check downstream DB/API latency because slow processing can create lag.
- **Consumer Health:** Check errors, retries, and whether consumers are alive.
- **Scaling:** Scale consumers only when partition count and workload allow useful parallelism.

### Q16. 🟢 [🌐] What if messages are duplicated?
**Answer:**
- **Expect Duplicates:** Assume duplicate delivery can happen in distributed messaging systems (At-Least-Once delivery).
- **Idempotency:** Make the consumer idempotent so processing the same event twice produces the same final result.
- **Unique IDs:** Use a unique event/message ID and store processed IDs where appropriate.
- **Other Checks:** Also check producer retries, consumer retries, acknowledgements, and offset handling.

### Q17. 🟢 [🌐] How to handle idempotency in retries?
**Answer:**
- **Idempotency Key:** Give the operation a unique idempotency key or request ID.
- **Store Status:** Store the key and resulting status/response so a retry can return the existing result.
- **Message Consumers:** For message consumers, use a unique event ID or business key and make the update safe to repeat.
- **Backoff:** Combine retries with timeouts and backoff; do not retry indefinitely.

### Q18. 🟢 [🌐] What if logs are not sufficient?
**Answer:**
- **Trace IDs:** First reproduce or correlate the issue using request/trace IDs.
- **Structured Logging:** Add structured logs around important business steps and dependency calls.
- **Useful Context:** Log useful context such as order ID, service, latency, and error type (avoid sensitive data).
- **Metrics & Traces:** Use metrics and distributed traces in addition to logs.
- **Targeted Logging:** For a production hotfix, add targeted logging rather than noisy logging everywhere.

### Q19. 🟢 [🌐] How to improve observability?
**Answer:**
- **Three Core Signals:** Use the three core signals: logs, metrics, and traces.
- **Dashboards:** Create dashboards for request rate, latency, errors, CPU, memory, DB, and dependency health.
- **Correlation:** Use correlation/trace IDs across services.
- **Alerts:** Set alerts on meaningful symptoms such as high error rate, p95 latency, and consumer lag.
- **Usability:** Make dashboards useful for both troubleshooting and trend detection.

### Q20. 🟢 [🌐] How do you implement distributed tracing?
**Answer:**
- **Trace ID Propagation:** Generate or propagate a trace ID across the request path.
- **Spans:** Create spans for service calls, DB calls, and important operations.
- **Standard Tools:** Use a tracing standard/tool such as OpenTelemetry and export traces to an observability platform.
- **Headers:** Ensure HTTP/message headers carry the trace context across service boundaries.
- **Latency Analysis:** Use traces to find where end-to-end latency is spent.

### Q21. 🟢 [🌐] What metrics do you monitor in production?
**Answer:**
- **Application:** Request rate, error rate, latency, and throughput.
- **JVM:** Heap, GC pauses, CPU, threads, and memory.
- **Database:** Query latency, connections, locks, and errors.
- **Microservices:** Dependency latency, timeouts, and circuit-breaker state.
- **Kafka:** Consumer lag, throughput, and processing errors.
- **Infrastructure:** CPU, memory, disk, and network.

### Q22. 🟢 [🌐] How to detect slow endpoints?
**Answer:**
- **Per-Endpoint Latency:** Monitor latency per endpoint rather than only overall service latency.
- **Tail Latency:** Use p95/p99 latency to identify slow requests affecting a meaningful portion of users.
- **APM:** Use tracing and APM to identify the slow code or dependency.
- **Baseline Comparison:** Compare endpoint latency before and after releases.
- **Alerting:** Set alerts for important endpoints when latency crosses an agreed threshold.

### Q23. 🟢 [🌐] What is p95 vs p99 latency?
**Answer:**
- **p95 Latency:** 95% of requests are at or below that latency; the slowest 5% are above it.
- **p99 Latency:** 99% of requests are at or below that latency; the slowest 1% are above it.
- **Tail Sensitivity:** p99 is more sensitive to tail latency and occasional very slow requests.
- **Importance:** Both are useful because average latency can hide bad user experiences.

### Q24. 🟢 [🌐] How do you debug GC pauses?
**Answer:**
- **GC Logs/Metrics:** Check GC logs/metrics for pause duration, frequency, and heap behavior.
- **Allocation Rate:** Check allocation rate and whether the application creates too many short-lived objects.
- **Heap Pressure:** Check old-generation growth and whether the heap is under pressure.
- **Profiling:** Take a Java Flight Recorder/profile when deeper analysis is needed.
- **Tuning:** Tune heap/GC only after understanding the allocation and memory pattern.

### Q25. 🟢 [🌐] How do you identify missing indexes?
**Answer:**
- **Slow Queries:** Find slow/high-frequency queries from DB monitoring or slow-query logs.
- **Inspect Columns:** Inspect the query's `WHERE`, `JOIN`, `ORDER BY`, and `GROUP BY` columns.
- **Execution Plan:** Use `EXPLAIN` or `EXPLAIN ANALYZE` to check whether the DB is doing a full table scan when an index could help.
- **Trade-offs:** Add an index only when it improves the workload; indexes also increase write/storage cost.
