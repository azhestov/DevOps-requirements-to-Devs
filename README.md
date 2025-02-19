**Documentation**

- A short description of the service (what it is, what it does, and why) at the beginning of `Readme.md`.
- Description of start/stop/safe restart/restart dependencies in `Readme.md`, including:
  - Direct dependencies (e.g., "I need to be restarted when the database restarts").
  - Known reverse dependencies (e.g., "If restarted, also restart service X").
- List of supported/required environment variables in `Readme.md`.
- API endpoints and service metrics documented in `Readme.md`.
- Repository link included in `Readme.md`.
- Link to documentation and API documentation in `Readme.md`.
- Contact information of the responsible developer in `Readme.md`.

(For a good example, try running `man curl` in a Linux terminal.)

---

**Monitoring**

- The service should support on-demand metrics and API endpoints.
- The service should be able to push metrics proactively.
- Health check status:
  - Minimum response: "OK/Not OK".
  - Recommended responses:
    - "Running, able to respond" (Liveness).
    - "Operational/Technical issues detected" (e.g., missing logs, configuration errors upon reload).
    - "All OK/Issues with dependent services" (e.g., database connection issues, backend unavailability, queue overflow) (Readiness).
- Metrics:
  - Minimum:
    - "Number of requests/operations per time interval".
    - "Average processing time per time interval".
    - JVM parameters (for Java applications).
  - Recommended:
    - "Processing time percentiles".
    - "Error count per interval".
    - Any other relevant operational metrics.
- The service should return different error codes for different error types.
- The service should return error codes conforming to standard HTTP status codes and Linux exit codes.

---

**Configuration and Logging**

- The service should be able to read parameters from environment variables.
- Environment variables should take precedence over configuration file parameters.
- All units (especially time) in configuration files should be standardized or explicitly defined (e.g., `Xd Yh` format, `Duration` for Java).
- The service should support a "request trace ID":
  - Accepting it, logging it, passing it to sub-requests, and forwarding it downstream.
- If a `User-Agent` is present, it should include at least the service name and version when calling other services.
- Logging capabilities:
  - Logs should be written to `STDOUT/STDERR`.
  - The service should log short stop/start/reload messages to `syslog`.
  - The service should be able to log to dedicated files (configurable).
  - Human-readable log format.
  - Separation of `access logs` and `error logs`.
  - Stack traces should be logged in `error.log`.
  - Errors should be logged with meaningful messages, not just stack traces.
  - System logs should be separated from business logic logs.
  - On demand, the service should log detailed transport logs, including:
    - Raw incoming request with parameters.
    - Processed request with applied transformations.
    - All headers, request body, and response details.
- If self-managed log rotation is necessary, it should be implemented and configurable (preferably with compression, or explicitly stated in `Readme.md` if missing).
- The service should support log level configuration:
  - Different log levels for different log types.
  - Dynamic log level adjustment without a restart (via environment variables, config reload, API request, or system signal).
- Logging destinations:
  - Support for remote storage (Elasticsearch, `rsyslog`).
  - Ability to write logs both locally and remotely.
  - Different log types should be storable in different locations.
- The service should support resource constraints (config/startup flags, documented in `Readme.md`).

---

**Build and Operation**

- The service should be built as a single deployable entity (`package/archive/venv/container`, etc.).
- The deployable entity should include `Readme.md`, ideally with its content available via `man`.
- All dependencies should be listed with fixed versions (at least `>=` specified).
- The service should handle `SIGTERM` and other OS signals gracefully, allowing current tasks to complete before shutting down.
- The service should support horizontal scaling:
  - No race conditions or contention for exclusive resources.
- If multiple instances run on the same host, the service should either:
  - Prevent duplicate instances.
  - Support proper operation with multiple instances.
- If the service listens to queues or events, format changes should not cause crashes.
- Failed task processing should be retried `N` times (configurable interval and retry count).
- Unrecoverable failures should not be silently discarded but instead logged or sent to a `Dead Letter Queue`.
- The service should support re-processing `Dead Letter Queue` entries upon request.
- The service should not crash due to queue processing issues.
- The service should support an environment variable limiting the max number of processed tasks before clean shutdown (to prevent memory leaks).
- No hardcoded IPs, hostnames, service names, or ports (e.g., HTTP may not always be on port 80).
- Database and queue handling:
  - The service should support read replicas.
  - The service should support read-only replicas.
  - The service should detect the master instance.
  - The service should iterate through available/unavailable replicas.
  - The service should handle replica recovery.
  - On database/queue connection loss, the service should not crash but:
    - Return a meaningful error status.
    - Attempt reconnection at configurable intervals.
- If external load balancers are required, this must be explicitly mentioned in `Readme.md`.
- Inter-service communication:
  - The service should handle multiple identical services across different hosts.
  - The service should iterate through available/unavailable instances.
  - The service should handle recovery of previously unavailable instances.
  - If all instances are unavailable, the service should not crash but:
    - Return a meaningful error status.
    - Attempt reconnection at configurable intervals.
- If an external load balancer is required, this must be explicitly stated in `Readme.md`.

---

**Database Considerations** 

- Database migration plans should be prepared before deployment.
- For 24/7 services that cannot afford downtime:
  - Changes should follow one of two deployment strategies:
    - **"Database first, then code"**: Ensure old code functions with the new database.
    - **"Code first, then database"**: Ensure new code functions with the old database.
- If backward compatibility is required during deployment:
  - Transitional code should be written.
  - This code should either:
    - Be removed in the next deployment cycle.
    - Have dual versions prepared for sequential rollouts.
- Destructive changes (e.g., dropping columns/tables/schemas, mass deletions/updates) should be deployed in a separate phase **after** main updates and code releases.

