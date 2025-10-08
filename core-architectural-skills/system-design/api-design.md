# 🔌 API Design (REST, GraphQL, gRPC)

Design clear, evolvable APIs aligned with client needs and domain boundaries.

## Styles
- REST: Resource-oriented, HTTP semantics (verbs, status codes, HATEOAS optional)
- GraphQL: Client-driven queries, single endpoint, strong schema
- gRPC: Contract-first (Protocol Buffers), efficient binary, streaming support

## Cross-Cutting Principles
- Versioning: URI vs header; backward compatibility by default
- Pagination and Filtering: Cursor-based preferred; consistent limits
- Errors: Structured errors with codes; use standard HTTP semantics
- Idempotency: Keys for unsafe methods (e.g., POST with Idempotency-Key)
- Security: AuthN/Z (OAuth2/OIDC), scopes/claims, input validation
- Rate Limiting and Quotas; Caching (ETag/Cache-Control)
- Documentation: OpenAPI/Swagger; GraphQL SDL; protobufs + annotations
- Contract Testing: Prevent breaking changes

## GraphQL Specifics
- N+1 avoidance (dataloaders); schema design and deprecation
- Authorization per field/type where needed

## gRPC Specifics
- Unary, client/server/bidi streaming
- Interceptors for auth/observability; deadlines instead of timeouts

## REST Specifics
- Resource modeling; sub-resources; partial updates (PATCH)

## Checklist
- Define API guidelines; provide linters and CI checks
- Adopt contract-first design where possible
- Provide SDKs/examples and sandbox environments
