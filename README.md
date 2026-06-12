# Connect, Not Build — to AI

## Background

During 2024–2025, software enterprises invested heavily in developing various AI Agents, achieving significant results and riding the wave of the AI era. Now that we have tangible outcomes, it's time to look back, look ahead, and explore the next steps for enterprise software.

Looking back, we observed:

- **High R&D costs.** Early AI Agent SDKs were immature and iterated rapidly, forcing teams to switch SDKs mid-project or stay on outdated versions with extensive customizations, making it impossible to keep up with newer releases.

- **Inconsistent product capabilities.** Each module team built its own Agent independently, leading to inconsistent end-user experiences — some Agents had memory, others didn't, or two Agents' memories were not shared.

- **Inability to keep pace.** A team worked overtime last week to ship a memory feature, only for customers to ask this week: "Other products already have sub-Agent collaboration — why doesn't yours?"

- **Lack of cross-module collaboration.** Large software suites contain many modules, but the finance AI can only answer finance questions. To drill into supply chain analysis, users must switch to the supply chain AI and re-explain context from scratch — even when they have the necessary permissions.

- **Lack of external collaboration.** Users want to use general-purpose AI tools like Codex or Claude to extract data from large ERP systems, write complex Python data-cleansing logic, have AI present results in Excel, and after review meetings, have AI import the approved Excel results back into the financial budgeting module.

## Root Cause Analysis

Analyzing the current AI workflow and our previous efforts, we find that we essentially built two layers of capability for AI: one layer of **general AI capabilities** (memory, rate limiting, permissions, skill support, multi-agent orchestration, etc.), and another layer of **domain-specific capability wrappers** — making AI able to operate within a specific software domain. These domain capabilities are the original software's own features, simply wrapped so AI can **perceive, decide, and act**.

Regarding general AI capabilities, our first instinct was: all product lines and modules should unify on a single AI Agent platform to reduce redundant effort.

But how should domain-specific capabilities be wrapped? Exposing them to a unified AI platform couples them to a specific technology stack — perhaps LangChain's Python ecosystem, or Microsoft Agent Framework's .NET platform. This is reminiscent of the old enterprise software dilemma: betting on Java's Spring ecosystem vs. .NET's ASP.NET ecosystem.

We need a **loosely-coupled, widely-supported standard** to wrap domain-specific capabilities — standards like MCP and Agent Skills — allowing AI platforms to **connect** to the software's specific domain. This yields the following advantages:

- Stop customizing general AI platforms. Instead, connect to superior AI platforms and eliminate the cost of building general-purpose AI infrastructure.

- Because it's a connection, you naturally get the latest AI features. When major AI products release new capabilities today, your customers benefit immediately — no waiting for you to catch up.

- Because it's a connection, capabilities are consistent, memory is shared, internal and external data is interconnected, and AI can seamlessly collaborate across different software systems.

MCP (Model Context Protocol) is an open standard for connecting AI applications to external systems. https://modelcontextprotocol.io/

Agent Skills is a lightweight, open format for extending AI agent capabilities with specialized knowledge and workflows. https://agentskills.io/

Agent2Agent (A2A) Protocol is an open standard for enabling seamless communication and collaboration between AI agents. https://a2a-protocol.org/

## Approach

To allow AI Agents to operate within your specific domain, you need to expose the following capabilities to AI:

### **Perception**

Imagine you need to assemble IKEA furniture. First, you need to see the wood panels, screws, tools, instruction manual, etc. Mapped to AI and enterprise software, this means letting AI perceive the current environment — including but not limited to: what modules exist, what tools are available, what skills are offered, and where to find deeper domain knowledge bases.

Perception also includes "current context" information. For example, you're in a team chat interface and ask AI: "Summarize what everyone was just discussing." To enable this, AI must perceive "what the current chat is." This type of contextual awareness extends to the current user, current project, current screen, and more.

In MCP, this corresponds to **resources**. However, MCP resources are currently host-facing — AI cannot read them directly. The official recommendation is for MCP servers to provide dedicated tools for reading resources, with results annotated as [resource links](https://modelcontextprotocol.io/specification/2025-11-25/server/tools#resource-links).

MCP supports returning **instructions** to [introduce itself to AI](https://modelcontextprotocol.io/specification/2025-06-18/schema#initializeresult), enabling AI to quickly perceive the software's specific domain.

MCP resources can also expose "current context" information and support notifying the Host when resources change — so when a user switches from one chat group to another, AI can update accordingly. However, you still need to expose current context via tools, since AI cannot access resources directly by default.

### **Skills**

Continuing the IKEA analogy: you can see all the materials and tools, and you've read the manual, but you don't know how to use the power drill or when to use which type of screw. What you need is a "skill" — knowledge of how to correctly complete a specific step. Mapped to AI and enterprise software, skills tell AI: in this specific domain, what are the standard procedures, constraints, and best practices for completing a certain type of task.

The distinction between skills and tools: **tools are atomic operations** (query a record, submit a form), while **skills are the knowledge of orchestrating multiple tools to accomplish a business-meaningful task**. For example, "creating a purchase order" is not a single tool call — it's a sequence of steps: verify supplier qualifications, check budget balance, populate default tax rates, generate an approval workflow, and notify stakeholders. AI needs to know the order of these steps, preconditions, and exception-handling strategies. That is a skill.

Skills can be expressed in multiple ways. The Agent Skills specification provides a standardized format including input/output schemas, execution steps, preconditions, and other metadata, enabling AI to understand and execute complex multi-step tasks.

Another key characteristic of skills is **composability**. A high-level skill can reference lower-level skills, forming a hierarchical capability system. For example, "month-end closing" can orchestrate sub-skills like "accounts receivable reconciliation," "expense accrual," and "exchange rate adjustment," each of which orchestrates multiple tool calls. This hierarchy allows AI to make decisions at the appropriate level of abstraction rather than reasoning from raw API calls.

Skills should also include **domain constraints** — "what must not be done." For example, approved documents cannot have their amounts retroactively modified; inter-entity fund transfers must undergo compliance review. If these constraints are not explicitly communicated to AI via skills, AI may attempt prohibited operations, causing data errors or compliance risks. Embedding constraints in skill descriptions is more reliable than relying on tool-level validation alone, because AI can avoid illegal paths during the planning phase.

MCP itself does not have a native Skills concept. Its current **prompts** mechanism is not exposed to AI the way Skills are, nor does it support "progressive disclosure" like Skills do. Skills can include additional templates, scripts, and resources beyond the main document — MCP prompts currently lack this design.

### **Action**

With perception and skills, AI knows what exists in the environment and how things should be done. The final step is actual execution — action. Back to the IKEA analogy: action is picking up the power drill, tightening the screws, and fastening the panels together. Mapped to AI and enterprise software, action is AI invoking tools to produce real, side-effect-bearing changes to the system.

In MCP, actions correspond to **tools**. Each tool represents an atomic operation: creating a record, updating a status, triggering a workflow, calling an external service, etc. Tool design should follow several principles:

- **Explicit side-effect declarations.** AI needs to know which tools are read-only (query, search) and which modify system state (create, delete, approve). MCP tool definitions use annotations to indicate whether a tool is destructive (`destructiveHint`) or read-only (`readOnlyHint`), enabling AI to assess risk before execution and avoid irreversible operations at inappropriate times.

- **Idempotency and transaction safety.** Many enterprise operations involve critical data like finances and inventory. Tool design should account for safe retry behavior. If AI retries a "deduct inventory" call due to a network timeout, the system should not double-deduct. This is not an AI-layer problem but a tool-implementation guarantee — yet tool descriptions should inform AI which operations are idempotent and which require state checks before execution.

- **Permission and approval handoff.** Not all actions should take immediate effect. Some operations exceed the current user's permissions or surpass thresholds requiring manual approval. Tools should not simply reject these requests but return a "pending approval" intermediate state, allowing AI to inform the user: "Operation submitted, awaiting manager approval" — rather than throwing a generic error. This requires tool response structures that can express asynchronous states and guidance for next steps.

- **Confirmability of operations.** For high-risk operations (data deletion, large transfers, irreversible state changes), tools should support a "preview-then-confirm" two-step pattern. AI first calls a preview endpoint to show the impact scope, and only executes the actual change after user confirmation. MCP tools don't enforce this pattern, but software designers should adopt it as a best practice to prevent irreversible damage during automated execution chains.

Action also involves **cross-system collaborative execution**. When a business process spans multiple modules or independent systems, AI needs to coordinate tool calls across multiple MCP Servers. For example, "supplier payment" may require confirming receipt in the procurement module, creating a payment voucher in the finance module, and initiating a bank transfer through a banking API. The A2A protocol serves this scenario — it enables structured task delegation and state synchronization between Agents of different systems, so cross-system actions no longer depend on a single Agent possessing all tools, but are accomplished through inter-Agent collaboration for end-to-end business processes.

---

*This is not a code project. It is an architecture discussion project.*
