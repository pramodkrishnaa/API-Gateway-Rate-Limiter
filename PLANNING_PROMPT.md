## 1. PLAN-MODE INSTRUCTION

You must operate strictly in **Plan / Read-Only Mode** for this entire request. You are to use read-only tools to explore the codebase, analyze existing patterns, and formulate a detailed implementation plan. 

**Do NOT write, edit, or delete any files on disk, and do NOT execute any state-changing shell commands or code modifications.** Once your comprehensive implementation plan is generated, you must explicitly **STOP** and wait for human review and approval before taking any further action.

---

## 2. CONTEXT

You are acting as a senior software architect inspecting the codebase to design a rate-limiting mechanism for our public API gateway. 

The raw task request is:
> "Add a per-IP token bucket rate limiter to the public API gateway, configurable requests-per-minute, return 429 with Retry-After on limit exceeded"

The primary goal is to protect the public API gateway from abuse and traffic spikes by introducing a per-IP token bucket rate limiter. The rate limiter must allow operators to configure the target requests-per-minute (RPM) threshold. When a client exceeds their allocated token allowance, the gateway must drop the request with an HTTP status code `429 Too Many Requests` and include a `Retry-After` header indicating how long the client must wait before retrying.

---

## 3. FUNCTIONAL REQUIREMENTS

Please design the architecture to satisfy the following numbered functional requirements:

1. **Token Bucket Algorithm Implementation**: Implement a standard token bucket rate-limiting algorithm. Tokens must refill smoothly or lazily based on elapsed time calculated against the configured requests-per-minute (RPM) rate.
2. **Per-IP Rate Limit Enforcement**: Rate limits must be tracked and enforced individually per client IP address.
3. **Proxy-Aware Client IP Extraction `[INFERRED]`**: The mechanism must properly resolve the actual client IP address, handling reverse proxies or load balancers (e.g., inspecting standard headers like `X-Forwarded-For` or `X-Real-IP` while guarding against header spoofing).
4. **Configurable RPM Settings**: The requests-per-minute (RPM) threshold (and optional burst capacity `[INFERRED]`) must be configurable via application configuration files or environment variables without requiring code re-compilation.
5. **HTTP 429 Response Format**: When a client's bucket has insufficient tokens for an incoming request:
   - Reject the request immediately with HTTP status code `429 Too Many Requests`.
   - Provide a structured JSON error body explaining that the rate limit was exceeded `[INFERRED]`.
6. **Retry-After Header**: Include a standard `Retry-After` HTTP header on all HTTP 429 responses. The header value must accurately reflect the integer number of seconds (or dynamic timestamp) remaining until at least one token is refilled and a request can be served.
7. **Rate Limit Transparency Headers `[INFERRED]`**: Include standard rate-limit response headers on allowed requests (e.g., `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`) to inform compliant clients of their current quota state.
8. **Gateway Middleware Integration**: Embed the rate limiter early in the API gateway execution pipeline so rejected requests terminate before incurring downstream processing or database resource costs.

---

## 4. TECHNICAL CONSIDERATIONS

When formulating your plan, thoroughly evaluate the following architectural and technical aspects:

* **State Storage & Storage Backend**: Determine whether state should be stored in-memory (local to the process) or distributed (e.g., Redis / Memcached). If the API gateway runs multiple instances behind a load balancer, an in-memory store will enforce per-instance limits rather than global per-IP limits. Provide recommendations for both local and distributed strategies.
* **Concurrency & Atomicity**: Token consumption and bucket updates must be atomic per IP to prevent race conditions during high-concurrency request bursts from a single source.
* **Memory Management & Garbage Collection**: In-memory tracking of transient IP addresses can lead to memory leaks if idle buckets are never cleaned up. Plan for automatic eviction or Time-To-Live (TTL) mechanisms for inactive IP state.
* **Performance Impact & Latency**: The rate-limiting layer sits in the critical hot path of every incoming public request. The check must execute in low sub-millisecond time and avoid blocking operations.
* **Security & IP Spoofing**: Validate how IP addresses are extracted. If relying on `X-Forwarded-For`, ensure the gateway only trusts headers appended by trusted upstream proxies.

---

## 5. FILES AND AREAS TO INVESTIGATE FIRST

Before writing your plan, perform a thorough, read-only exploration of the codebase. Focus your investigation on finding:

1. **API Gateway Entry Points & Pipeline**: Search for middleware chains, route definitions, or request interceptors where global public requests enter the system.
2. **Configuration Modules**: Locate existing configuration parsers, environment variable loaders, or settings files to see how parameters (like RPM thresholds) should be exposed and validated.
3. **Existing Caching or Data Store Integration**: Search for existing abstractions or wrappers for memory stores, Redis, or key-value caches that could host the token bucket states.
4. **Error Handling & Response Utilities**: Find standard HTTP response helpers or error-handling components to ensure HTTP 429 responses conform to existing system error schemas.
5. **Testing Frameworks & Utilities**: Look for existing unit and integration test setups for gateway middleware to understand how to write tests for time-dependent algorithms.

---

## 6. EDGE CASES AND FAILURE MODES

Your proposed plan must address how the system handles the following failure modes and edge cases:

* **Storage Store Unavailability / Failures**: If the state store (e.g., Redis) is unreachable or times out, decide whether the gateway should fail-open (allow requests to preserve availability) or fail-close (block traffic for security).
* **Clock Skew & Timestamp Resets**: Handling potential system clock drift or NTP adjustments when calculating elapsed time for token refills.
* **IPv4 vs IPv6 Addresses**: Normalizing IP address formats (e.g., handling IPv6 subnets or expanded vs compressed notations) so single actors cannot bypass limits using IP representation variations.
* **Burst Traffic at Boundary**: Managing exact boundary conditions when a client sends simultaneous concurrent requests with 0 tokens available.
* **Zero or Invalid Configuration**: Fallback behavior if configured RPM is set to 0, negative values, or malformed strings.

---

## 7. OPEN QUESTIONS

If your exploration of the codebase uncovers ambiguities, explicit assumptions, or structural constraints, detail them in an **Open Questions & Assumptions** section within your plan. Specifically flag assumptions regarding:
- Trusted proxy setups and IP resolution logic.
- Choice between distributed storage (e.g., Redis) vs in-memory local caching for your initial implementation phase.
- Scope of rate limiting (global across all gateway routes vs path-specific limits).

---

## 8. DELIVERABLE FORMAT

Your response must be a comprehensive, structured implementation plan containing the following sections:

1. **Summary of Proposed Architecture**: High-level overview of the chosen token bucket design, state store, and middleware positioning.
2. **Exploration Findings**: Brief notes on relevant existing files, middleware patterns, and configuration conventions found during your repository search.
3. **Phased Implementation Plan**: A ordered, step-by-step list of proposed file creations and edits (specify target filenames, components to modify, and purpose of change — do NOT produce full file diffs or full code blocks).
4. **Testing & Verification Strategy**: A concrete testing approach covering unit tests for token bucket logic (including time simulation), middleware integration tests, and edge case coverage.
5. **Risks, Rollback & Mitigation**: Operational risks introduced by this change and strategies for safe rollout and feature flag toggling.
6. **Open Questions & Assumptions**: List of decisions requiring human confirmation before coding begins.

*Reminder: This deliverable must be a text plan only. Do not execute any file creation or modification commands.*

---

## 9. PLAN-MODE INSTRUCTION (RESTATEMENT)

**OPERATE IN READ-ONLY PLAN MODE ONLY.** Inspect the repository using read-only tools, compose the detailed implementation plan specified above, and **STOP** immediately without editing any files or writing code to disk. Wait for explicit human confirmation before proceeding.