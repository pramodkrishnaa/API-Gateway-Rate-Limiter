# forge-api-gateway-rate-limiter

Scaffolding repo created by Forge (QuadGen Agentic Workforce) on 2026-09-02.

## Purpose
Implement a per-IP token bucket rate limiter for the public API gateway. The rate limiter must support configurable requests per minute and respond with HTTP 429 and a Retry-After header when limits are exceeded.

## How to use this
1. Open a new Claude Code session in the repository you actually want to implement this in.
2. Enable Plan Mode.
3. Paste the full contents of `PLANNING_PROMPT.md` into the session.
4. Review Claude Code's proposed plan before approving any implementation.

## Source
- Input type: command
- Original input: Add a per-IP token bucket rate limiter to the public API gateway, configurable requests-per-minute, return 429 with Retry-After on limit exceeded
- Drafted by: Forge, provider=gemini-3.6-flash, 2026-09-02T16:03:35
