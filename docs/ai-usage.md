# AI Agent Usage Guide

This project ships with five AI agent prompts in the `agents/` directory. Each is a structured prompt you paste into your AI tool (Claude, GitHub Copilot Chat, Cursor, etc.) along with your specific input.

**Start here**: use the **Orchestrator** agent (`agents/orchestrator.md`) to implement a complete feature end-to-end. It runs the other agents in the correct order with quality gates between steps. Use the individual agents only when you need a specific step in isolation.

## How agents work

Each agent file defines a **role**, a **goal**, **constraints**, and an `{{input}}` placeholder. You fill in the placeholder and send it to your AI tool. The agent's constraints ensure the generated code fits this project's stack without modification.

---

## Agent: Orchestrator (`agents/orchestrator.md`)

**Use when**: You want to implement a feature completely — design, code, tests, and review — in one go.

**What it does**: Runs a four-step pipeline, delegating to the other agents in sequence, and enforcing quality gates between steps:

```
Step 0: Classify scope (new feature vs. new microservice)
Step 1: Generate implementation  →  agents/rest-api.md
Step 2: Generate tests           →  agents/test-generator.md
Step 3: Code review              →  agents/code-review.md  (loops until no Critical/Major issues)
Step 4: Delivery summary
```

**How to use**:
1. Open `agents/orchestrator.md`
2. Copy the entire content
3. Replace `{{description}}` with your feature description
4. Paste into your AI tool — the agent will walk you through each step

**Example input**:
```
Implement a Product management feature.
A Product has: id (Long), name, sku (unique, max 50 chars), price (BigDecimal, positive), stockQuantity (int, min 0).
Endpoints: create, get by id, list all (paginated), update price, soft delete.
Business rule: price cannot be below 0.01. Stock cannot go negative.
```

**When to use the orchestrator vs. individual agents**:
- Use the orchestrator for new features, new entities, or new endpoints.
- Use individual agents when you want one specific output: generate tests for existing code, review a PR, or design a service independently.

---

## Agent: REST API Generator (`agents/rest-api.md`)

**Use when**: You need a new resource (entity + CRUD endpoints).

**What it generates**: Entity, Repository, Service, Controller, request/response DTOs, mapper, and integration tests — all wired together.

**How to use**:
1. Open `agents/rest-api.md`
2. Copy the entire content
3. Replace `{{description}}` with your feature description
4. Paste into your AI tool

**Example input**:
```
Generate a REST API for managing Product inventory.
A Product has: id (Long), name, sku (unique), price (BigDecimal), stockQuantity (int).
Endpoints: create product, get by id, list all (paginated), update price, delete.
```

**Example with more business rules**:
```
Generate a REST API for an Order management system.
An Order has: id, customerId (Long FK), status (PENDING/CONFIRMED/CANCELLED), items (list of OrderItem), createdAt.
An OrderItem has: productId, quantity, unitPrice.
Business rules:
- An order cannot be cancelled if status is CONFIRMED
- Stock must be checked before confirming an order
Endpoints: place order, confirm order, cancel order, get order by id, list orders by customer.
```

---

## Agent: Test Generator (`agents/test-generator.md`)

**Use when**: You have a service or controller and need tests written for it.

**What it generates**: Unit tests for the service (with Mockito), controller tests (with `@WebMvcTest` + `MockMvc`), and integration tests (extending `BaseIntegrationTest`).

**How to use**:
1. Copy `agents/test-generator.md`
2. Replace `{{code}}` with the class(es) you want tested
3. Send to your AI tool

**Example input**:
```
Generate tests for the following OrderService:

[paste your OrderService.java content here]
```

**Tip**: Paste one class at a time for best results. Start with the service, then the controller.

---

## Agent: Code Review (`agents/code-review.md`)

**Use when**: You want a second opinion on code before committing or opening a PR.

**What it checks**: Architecture violations, Spring Boot anti-patterns, N+1 queries, missing validation, security issues, and code quality.

**How to use**:
1. Copy `agents/code-review.md`
2. Replace `{{code}}` with the code to review
3. Send to your AI tool

**Example input**:
```
Review this code:

[paste your Java class here]
```

The agent will report each issue with severity (`Critical`, `Major`, `Minor`, `Suggestion`) and a concrete fix.

---

## Agent: Microservice Designer (`agents/microservice.md`)

**Use when**: You're starting a brand new service and want to think through the design before writing code.

**What it produces**: Bounded context, domain model, API design, package structure, dependencies, and key architectural decisions.

**How to use**:
1. Copy `agents/microservice.md`
2. Replace `{{description}}` with the service purpose and requirements
3. Send to your AI tool

**Example input**:
```
Design a Payment microservice.
It processes payments for orders from the Order service.
It must support Stripe and PayPal as payment providers.
It must store payment history and allow querying payments by order ID.
It should emit a PaymentCompleted event that the Order service listens to.
```

---

## Agent skills

The `agents/skills/` directory contains reusable constraint documents:

| File | Purpose |
|---|---|
| `spring-boot.md` | Spring Boot conventions enforced across all agents |
| `testing.md` | Testing framework rules and naming conventions |
| `docker.md` | Docker and Docker Compose conventions |

You can reference these in a custom agent prompt or prepend them to any request to add context:

```
[paste agents/skills/spring-boot.md]

Now generate a ProductService for...
```

---

## Claude Code (MCP servers)

If you use **Claude Code** as your AI tool, configure the MCP servers in `agents/mcp/servers.json`. This file tells Claude Code which external servers to connect to.

To install MCP servers:
```bash
# GitHub MCP server (for PR and issue context)
claude mcp add github

# Filesystem MCP (for cross-file awareness)
claude mcp add filesystem --scope project
```

See the [Claude Code MCP documentation](https://docs.anthropic.com/en/docs/claude-code/mcp) for full setup instructions.

---

## Tips for better results

- **Be specific about business rules** — the more constraints you give, the more correct the output.
- **Paste existing related code** alongside the agent prompt to keep the AI consistent with your current style.
- **Iterate** — generate a first version, then use the Code Review agent to check it, then the Test Generator agent.
- **Chain agents** — design with Microservice → generate with REST API → test with Test Generator → review with Code Review.
