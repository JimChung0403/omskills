---
name: Legacy Java Architect
description: Analyze legacy Java/Java EE project architecture — protocols, upstream/downstream interfaces, data model, per-endpoint business logic tracing, and cross-endpoint workflow assembly. Use this skill when users want to understand a legacy Java project's communication protocols, find upstream/downstream system boundaries, or trace business logic per endpoint. Trigger on phrases like "analyze Java project", "trace endpoint logic", "find protocols", "upstream downstream", "EJB analysis", "system boundary", or any request to reverse-engineer a Java EE application's architecture and data flow
---

# Legacy Java Architect

Analyze a legacy Java/Java EE project (potentially 10-20+ years old) through five phases:

1. **Protocol Scan** — what protocols and endpoints does this project expose
2. **Upstream/Downstream** — what external systems does it connect to
3. **Data Model** — Entity relationships, table structures, ER diagram
4. **Endpoint Trace** — per-endpoint full business logic, computation, and data flow tracing
5. **Workflow Assembly** — cross-endpoint business process reconstruction

## Workflow

```
Phase 1 ──→ Phase 2 ──→ Phase 3 ──→ Phase 4 ──────→ Phase 5
Protocol    Upstream/    Data        Endpoint        Workflow
Scan        Downstream   Model       Trace           Assembly
(1 time)    (1 time)     (1 time)    (per endpoint)  (1 time)

Reads:      Reads:       Reads:      Reads:          Reads:
  (none)      01-own-      01-own-     01-own-         all endpoints/
              protocols    protocols   protocols       shared/
                           02-upstream 02-upstream     03-data-model
                           -downstream 03-data-model

Produces:   Produces:    Produces:   Produces:       Produces:
  01-own-     02-upstream  03-data-   shared/          workflows/
  protocols   -downstream  model      endpoints/       05-workflow-
              02-system-              {METHOD}-         summary
              overview                {path}.md
```

### Phase 1: Protocol Scan

Read `references/phase1-protocol-scan.md` for the complete procedure.

Scan 9 types of inbound protocols the project exposes to callers, plus global project configuration:
- REST API (Spring MVC + JAX-RS)
- SOAP Web Service (@WebService)
- Servlet / Filter (web.xml + @WebServlet)
- EJB Session Bean (@Stateless/@Stateful/@Singleton + @Remote/@Local, including EJB 2.x)
- Message-Driven Bean / JMS Consumer (@MessageDriven, @JmsListener)
- Global config: build dependencies (pom.xml), security, Spring XML beans, app server settings
- WebSocket (@ServerEndpoint)
- Scheduled Tasks (@Scheduled, @Schedule, Quartz)
- RMI
- gRPC

EJB multi-protocol exposure: a single @Stateless bean may also carry @WebService and @Path — treat it as one service unit with multiple exposure methods.

### Phase 2: Upstream/Downstream

Read `references/phase2-upstream-downstream.md` for the complete procedure.

Requires Phase 1 complete. Scan 9 types of outbound connections:
- Database (JDBC, JNDI DataSource)
- REST Client (RestTemplate, WebClient, HttpClient, Feign)
- SOAP Client (generated stubs, wsimport)
- Message Queue producer (JmsTemplate, KafkaTemplate, RabbitTemplate)
- Cache (Redis, EhCache, Memcached)
- LDAP
- Email (SMTP)
- File / FTP / SFTP
- Other (Elasticsearch, MongoDB, Socket)

Produces a Mermaid system boundary diagram.

### Phase 3: Data Model

Read `references/phase3-data-model.md` for the complete procedure.

Requires Phase 1 & 2 complete. Build the complete data model:
- JPA/Hibernate @Entity scanning with field structures
- Relationship mapping (@ManyToOne, @OneToMany, @ManyToMany, @JoinColumn)
- EJB 2.x CMP Entity Bean and CMR fields
- Hibernate XML mapping (.hbm.xml)
- SQL-based table discovery (tables accessed without Entity mapping)
- DTO/VO/Request/Response structure scanning and Entity ↔ DTO mapping
- Cross-reference Entity ↔ Table ↔ SQL ↔ DTO

Produces a Mermaid ER diagram.

### Phase 4: Endpoint Trace

Read `references/phase4-endpoint-trace.md` for the complete procedure.

Requires Phase 1, 2 & 3 complete. For each endpoint from Phase 1, trace 9 dimensions:

1. **Entry point** — method signature, parameters, return type
2. **Call chain** — Controller/EJB → Service → DAO, max 10 layers deep; includes event/async branches
3. **Data access** — SQL (JPA/Hibernate/MyBatis/JDBC), Stored Procedures, tables accessed; cross-references Phase 3 ER model
4. **External calls** — REST/SOAP/MQ calls to downstream systems
5. **Business logic** — conditions, computations, validations, data transformations, config-driven logic, rule engines
6. **Error handling** — try-catch scope, exception types, global handlers
7. **Interceptors / AOP** — EJB @AroundInvoke chain, Spring @Aspect/@Around/@Before/@After
8. **Security** — @RolesAllowed, @PreAuthorize, programmatic checks
9. **Transactions** — Spring @Transactional, EJB CMT (@TransactionAttribute), BMT (UserTransaction), cross-EJB propagation chain

Injection resolution covers: @Autowired, @Inject, @Resource (Spring), @EJB (EJB 3.x), JNDI lookup / ServiceLocator (EJB 2.x), XML wiring.

**Shared services**: Classes injected 3+ times are analyzed once in `shared/` and referenced by endpoint traces.

**Pacing**: analyze 3-5 endpoints per session. Each endpoint gets its own file.

### Phase 5: Workflow Assembly

Read `references/phase5-workflow-assembly.md` for the complete procedure.

Requires Phase 4 substantially complete (core endpoints analyzed). Assemble cross-endpoint business processes:
- MQ producer ↔ consumer matching (by Queue/Topic name)
- Event publisher ↔ listener matching (by Event class type)
- Internal REST/SOAP/EJB call chaining
- DB write → read indirect linking (same Table across endpoints)
- Timer/scheduler integration into process chains
- Orphan endpoint identification

Produces Mermaid sequence diagrams for each business workflow.

## Memory Bank

Write all results to `analysis-memory/` for cross-session persistence:

```
analysis-memory/
├── progress.md                  ← Read this FIRST every session
├── endpoint-checklist.md        ← Phase 1 generates, Phase 4 updates per endpoint
├── 01-own-protocols.md          ← Phase 1 output
├── 02-upstream-downstream.md    ← Phase 2 output
├── 02-system-overview.md        ← Phase 2 Mermaid diagram
├── 03-data-model.md             ← Phase 3 output (Entity + ER diagram)
├── shared/                      ← Phase 4 pre-step (shared services)
│   ├── AuditService.md
│   └── ...
├── endpoints/                   ← Phase 4 output (one file per endpoint)
│   ├── GET-api-users.md
│   ├── EJB-OrderService-createOrder.md
│   ├── SOAP-getAccount.md
│   └── ...
├── workflows/                   ← Phase 5 output (one file per workflow)
│   ├── order-creation.md
│   └── ...
└── 05-workflow-summary.md       ← Phase 5 summary + orphan endpoints
```

**Rules**:
- At session start: read `progress.md`. If it exists, resume from last incomplete phase. If not, create it and start from Phase 1.
- After each phase/endpoint/workflow: write results to the corresponding file and update `progress.md`.
- When user says "continue": read `progress.md` to determine current phase, then follow the continue behavior below.

**Continue behavior by phase**:
- Phase 1-3, 5: Automatically resume the incomplete phase.
- Phase 4 (endpoint trace): Read `endpoint-checklist.md`, display a summary table to the user showing completion stats and the next pending endpoints, then ask the user which endpoint(s) to analyze. Do NOT auto-pick — let the user choose or confirm. If the user says "continue" again without picking, analyze the next endpoint by priority order (P0 > P1 > P2 > P3).

## User Commands

| User says | Action |
|---|---|
| `start` | Start Phase 1 — read `references/phase1-protocol-scan.md` and execute |
| `Phase N` | Read the corresponding phase reference and execute |
| `analyze {endpoint}` | Read `references/phase4-endpoint-trace.md`, trace the specified endpoint. Mark it P0 in checklist |
| `continue` | Read `progress.md`. If in Phase 4: show `endpoint-checklist.md` summary and ask user to pick. Otherwise: resume next incomplete phase |
| `status` | Read and summarize `progress.md`. If in Phase 4: also show `endpoint-checklist.md` completion stats |
| `overview` | Generate system architecture overview from collected data |

## Output Language

Detect the user's conversation language. Write all reports in that language. Keep technical terms in English with explanation on first occurrence.

## Quality Rules

- All conclusions must cite source code location (file:line)
- Never guess — all analysis must be based on actual code
- Mark uncertain items as "⚠ to be confirmed", never skip them
- Protocols not found: record "not detected", never fabricate
- Include Mermaid diagrams: `graph TD` for system overview, `erDiagram` for data model, `flowchart TD` for call chains, `flowchart LR` for data flow, `sequenceDiagram` for workflows
