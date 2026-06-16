# Connect, Not Build — to AI

## Background

During 2024–2025, software enterprises invested considerable effort into developing various AI Agents and achieved significant results, catching the wave of the rapidly evolving AI era. Now that we have fruitful outcomes, it is time to look back, look ahead, and explore the next steps for software enterprises.

Looking back, we observed:

- High AI R&D costs, primarily because early AI Agent SDKs were immature and iterated rapidly, forcing teams to switch SDKs mid-project or stay on very early versions with extensive customizations, making it impossible to keep up with newer SDK releases.

- Inconsistent product features. Each module team developed its own Agent independently, leading to inconsistent end-user experiences — for example, some Agents might have memory while others do not, or two Agents' memories are not shared.

- Product features could not keep pace with the rapid rhythm. A team worked overtime last week to finally ship a memory feature, only for customers to ask this week: other products already have sub-Agent collaboration, why does yours not?

- Lack of cross-module collaboration. Large software suites contain numerous modules, but the finance AI can only answer finance questions. To drill down into supply chain analysis, even when the user has permission, they must switch to the supply chain AI and re-explain everything.

- Products also lacked external collaboration capability. Users expect to use general-purpose AI tools such as Microsoft Copilot and OpenWork to extract data from large ERP systems, have AI write complex Python cleansing logic, then have AI present the results clearly in Excel, and after meeting discussions, have AI import the reviewed Excel results back into the financial budgeting module.

## Reflection and Direction

Analyzing the current AI working model and our previous efforts, we found that we essentially did two parts of work to provide capabilities for AI. One part is general AI capabilities — including but not limited to memory, rate limiting, permissions, SKILL support, multi-agent orchestration, and so on. The other part is the "wrapping" of the software enterprise's own unique capabilities — that is, enabling AI to operate within a specific software domain. These capabilities are the original software's own features; we merely wrap them so AI can **perceive, decide, and act**.

### General Capability Layer: Connect, Not Build

Regarding general AI capabilities, our first reaction was: all product lines and modules should unify on a single underlying AI Agent platform to reduce internal friction.

But the deeper question is: should this unified platform also be built in-house? Memory, multi-agent collaboration, context management, rate limiting, security — each of these capabilities is evolving rapidly, and building in-house means perpetually playing catch-up. A more pragmatic strategy is to **connect** to excellent AI platforms that are already doing this work, rather than reinventing one ourselves.

- Stop customizing general AI platforms; instead, fully connect to superior AI platforms and cut the cost of developing general-purpose AI infrastructure.

- Because it is a connection, you naturally enjoy the latest AI features. When major-vendor AI products release new capabilities today, your customers benefit immediately — rather than waiting for you to catch up.

- Likewise, because it is a connection, capabilities are consistent, memory is shared, internal and external data are interconnected, and AI can collaborate effortlessly across different software systems.

Of course, mainstream AI platforms currently offer incomplete support for enterprise scenarios — requirements such as multi-tenant isolation, fine-grained permissions, and compliance auditing are not yet adequately covered. During this transitional phase, existing in-house AI Agent platforms remain an effective complement, but their architectural direction should align with the "connect" approach: general capabilities should gradually be ceded to external platforms, while the in-house part focuses on the enterprise governance needs that external platforms do not yet cover, rather than duplicating foundational AI capabilities.

### Domain Capability Layer: Open Standards Are Needed

So how should domain-specific capabilities be wrapped? If these capabilities are "exposed" to a unified in-house AI platform, they become coupled to a specific technology stack — perhaps LangChain's Python ecosystem, or Microsoft Agent Framework's .NET platform. This is very much like the old enterprise software gamble of betting on either Java's Spring ecosystem or .NET's ASP.NET ecosystem.

We need to find a loosely-coupled, widely-supported standard to wrap domain-specific capabilities, so that any AI platform can **connect** to the software's specific domain.

### Selection: MCP and Agent Skills

MCP (Model Context Protocol) is an open standard for connecting AI applications to external systems. https://modelcontextprotocol.io/

Agent Skills is a lightweight, open architecture that can be used to enhance AI agent capabilities with various specialized knowledge and workflows. https://agentskills.io/

Agent2Agent (A2A) Protocol is an open standard designed to enable seamless communication and collaboration between AI agents. https://a2a-protocol.org/

We recommend MCP and Agent Skills as the current preferred standards, for the following reasons:

**Broad ecosystem support.** MCP is natively supported by mainstream enterprise AI products such as Microsoft Copilot, ChatGPT Enterprise, Claude Cowork, and Gemini Enterprise, and Agent Skills has also been adopted by products such as Claude and GitHub Copilot. Choosing these standards means your customers can connect directly to your software using any mainstream AI product, rather than only through your in-house AI gateway.

**Technological decoupling.** These standards interact with business systems through a standardized communication layer (JSON-RPC / HTTP), without binding to a specific programming language or runtime framework. Whether your business system is implemented in Java, .NET, Go, or any other language, it does not affect how AI connects to it.

**Evolutionary resilience.** Standards change — today it is MCP, and tomorrow a new protocol may emerge. But if support for these standards is built as a thin adapter layer — rather than embedding business logic in protocol code — then when standards evolve, the impact is confined to the adapter, and core business capabilities need not change. This is the classic adapter pattern: **capabilities are assets; protocols are merely adapter layers.**

Regarding A2A, the problem it solves (cross-system Agent collaboration) is real, but the standard itself is still early-stage with limited ecosystem support. We recommend reserving integration points via a loosely-coupled architecture, and formally adopting it once it matures.

## Approach

To enable AI Agents to enter your specific domain, we need to expose the following capabilities to AI:

### **Perception**

Imagine you now need to assemble a piece of IKEA furniture. Do you not first need to see the wood panels, screws, tools, instruction manual, and so on? Mapped to AI and enterprise software, this means letting AI perceive the current environment — including but not limited to: what modules exist, what tools and skills are available, and where to find deeper domain knowledge bases.

Perception also includes "current" information. For example, you are in the company's collaboration chat interface and ask AI: summarize what everyone was just talking about. To implement this, you must let AI perceive "what the current chat is." There are many such perceptions of current information: the current user, the current project, the screen currently being operated on.

In MCP this corresponds to resources, but currently MCP resources are host-facing — AI cannot read them directly. The official recommendation is for MCP to provide a dedicated resource-reading tool, with the returned result annotated as a [resource](https://modelcontextprotocol.io/specification/2025-11-25/server/tools#resource-links).

MCP supports returning instructions to [introduce itself to AI](https://modelcontextprotocol.io/specification/2025-06-18/schema#initializeresult); using this mechanism lets AI quickly perceive the software's specific domain.

MCP resources can likewise be used to expose "current" information, and they also support notifying the Host when resources change — so that when a user switches from one chat group to another, it can update promptly. But you still need to use tools to expose current information, since by default AI cannot directly access resources.

### **Skills**

Continuing the IKEA analogy: you have seen all the materials and tools, and you have read the manual, but you do not know how to use the power drill or when to use which type of screw. What you need now is a "skill" — the knowledge of how to correctly complete a specific step. Mapped to AI and enterprise software, a skill tells AI: in this specific domain, what are the standard procedures, constraints, and best practices for completing a certain type of task.

The distinction between skills and tools is that tools are atomic operations (query a record, submit a form), whereas skills are the knowledge of orchestrating multiple tools to complete a business-meaningful task. For example, "create a purchase order" is not a single tool call but a series of steps: verify supplier qualifications, check budget balance, populate default tax rates, generate an approval workflow, and notify relevant people. AI needs to know the order of these steps, preconditions, and exception-handling strategies. That is a skill.

Skills can be expressed in multiple ways. The Agent Skills specification provides a standardized description format that includes input/output schemas, execution steps, preconditions, and other metadata, enabling AI to understand and execute complex multi-step tasks.

Another key characteristic of skills is composability. A high-level skill can reference lower-level skills, forming a hierarchical capability system. For example, the "month-end closing" skill can orchestrate sub-skills such as "accounts receivable reconciliation," "expense accrual," and "exchange rate adjustment," each of which in turn orchestrates multiple tool calls. This hierarchy allows AI, when facing complex business scenarios, to not have to reason from the lowest-level API calls, but instead make decisions at an appropriate level of abstraction.

Skills should also include domain constraints — that is, "what must not be done." For example, approved documents are not allowed to have their amounts retroactively modified, and cross-entity fund transfers must undergo compliance review. If these constraints are not explicitly communicated to AI in the form of skills, AI may attempt non-compliant operations, leading to data errors or compliance risks. Embedding constraints in skill descriptions is more reliable than relying on tool-level validation, because AI can avoid illegal paths during the planning phase.

Currently SKILL is an independent specification; MCP itself has no direct SKILL concept. The currently provided prompts mechanism is not exposed to AI the way SKILLs are, nor does it have the "progressive disclosure" that SKILLs have. Beyond the main document, a SKILL can also contain additional templates, scripts, and so on, whereas current MCP prompts lack this design.

### **Action**

With perception and skills, AI already knows what is in the environment and how to do things. The final step is actual execution — that is, action. Back to the IKEA analogy: action is picking up the power drill, tightening the screws, and fastening the panels together. Mapped to AI and enterprise software, action is AI invoking tools to produce real, side-effect-bearing changes to the system.

In MCP, actions correspond to tools. Each tool represents an atomic operation: creating a record, updating a status, triggering a workflow, calling an external service, and so on. Tool design should follow several principles:

- Explicit side-effect declarations. AI needs to know which tools are read-only (query, search) and which modify system state (create, delete, approve). MCP's tool definitions use annotations to mark whether a tool is destructive (`destructiveHint`) or read-only (`readOnlyHint`); AI uses this to perform risk assessment before execution, avoiding irreversible operations at inappropriate times.

- Idempotency and transaction safety. Many operations in enterprise software involve critical data such as funds and inventory. When designing tools, you should consider the safety of repeated calls. If AI retries a "deduct inventory" call once due to a network timeout, the system should not double-deduct. This is not a problem the AI layer can solve but a guarantee the tool implementation layer must provide; nonetheless, tool descriptions should tell AI which operations are idempotent and which require a state check before execution.

- Permission and approval handoff. Not all actions should take effect immediately. Some operations exceed the current user's permissions, or surpass a threshold that requires manual approval. In these scenarios, tools should not simply reject but return a "pending approval" intermediate state, letting AI inform the user that "the operation has been submitted, awaiting manager approval," rather than simply reporting an error. This requires the tool's return structure to be able to express asynchronous states and guidance for subsequent actions.

- Confirmability of operations. For high-risk operations (data deletion, large transfers, irreversible state changes), tools should support a "preview-then-confirm" two-step model. AI first calls the preview interface to show the scope of the operation's impact, then executes the actual change after user confirmation. MCP's tools do not enforce this pattern themselves, but when designing tools, software should adopt it as a best practice, to avoid irreversible consequences within AI's automated execution chains.

Action also involves cross-system collaborative execution. When a business process spans multiple modules or even multiple independent systems, AI needs to coordinate tool calls across multiple MCP Servers. For example, "supplier payment" may require first confirming receipt in the procurement module, then creating a payment voucher in the finance module, and finally initiating a transfer via the banking interface. The A2A protocol plays a role in this scenario — it allows structured task delegation and state synchronization between Agents of different systems, so that cross-system actions no longer depend on a single Agent holding all the tools, but are accomplished through inter-Agent collaboration to complete end-to-end business processes.

## Implementation Path

1. **Identify core domain capabilities.** Map out the business capabilities across software modules that AI needs to operate, and clarify which are perception, which are skills, and which are actions.

2. **Build a capability model and adapter layer.** Separate the core implementation of business capabilities from protocol exposure. The protocol layer (MCP Server, SKILL files) exists as a thin adapter and does not carry business logic.

3. **Prioritize connecting one mainstream AI platform for validation.** Choose an AI product that already supports MCP (such as Copilot, Claude, or the company's own AI product that already supports MCP and SKILLS) for end-to-end validation, to quickly obtain feedback.

4. **Extend the in-house AI platform via middleware, plugins, and similar means.** If the in-house AI platform grew out of an open-source project, you should **strongly avoid directly modifying the platform's code**. These platforms typically offer third-party extension mechanisms — for example, LangChain provides [middleware](https://docs.langchain.com/oss/python/langchain/middleware/overview), and OpenWork is built on OpenCode, which offers [plugins](https://opencode.ai/docs/plugins/). This lets the in-house AI platform keep up with the latest versions quickly while retaining the enterprise governance layer as a supplement.

5. **Establish a governance baseline.** Unify the authentication and authorization gateway, operation audit logs, and data masking policies, to ensure security and compliance when AI connects to business systems.

## Appendix

### Known Controversies and Risks of Current Standards

- The MCP protocol currently has a "prompt explosion" controversy — when connecting to a large number of MCP Servers, tool descriptions significantly occupy the context window. Mainstream products have already mitigated this through approaches such as "progressive disclosure" or dynamic search and injection. Some vendors prefer using CLI as an alternative; CLI is naturally AI-friendly and does not occupy prompt space. This does not affect the core architectural choice, because CLI can likewise serve as a kind of adapter layer for capabilities.

- The A2A standard's prospects are currently unclear, with low community adoption. We recommend observing while reserving architectural integration points. That said, we still consider it a future direction worth close attention: enterprise-grade AI Agents face a very large number of Skills and MCP Tools, and given that LLM-level technology is unlikely to break through context-length limits in the near term, independent Agents per module is an inevitable trend. Coupled with the trend of Agents becoming increasingly autonomous, the demand for collaborative communication between Agents will grow ever more prominent — and this is precisely the core problem A2A is meant to solve.

- Regarding Ontology. In enterprise-grade AI there is another line of thought — represented by Palantir — that first builds a unified object semantic model (entities, relations, actions, invariants), through which AI performs tasks via typed, governed actions, with governance built into the model. Its key difference from MCP's flat approach is: ontology is "semantic-model-first," whereas MCP is "protocol-first, incrementally exposed." The two are not opposed — ontology can serve as the **source of truth for domain capabilities**, with MCP as its **open exposure layer**; objects generate flat tools, which neatly echoes this article's thesis that "capabilities are assets, protocols are adapter layers." In practice: enterprises that already have rich object models can manage their assets directly as an ontology and expose them via MCP; those that have not yet built such models should consciously model at points where domain richness concentrates, to avoid assets solidifying into a flat-function form. The cost of ontology is high modeling effort and possible lock-in to proprietary platforms — a trade-off to weigh.

### Further Reading

For a detailed technical exposition of the architectural idea that "capabilities are assets, protocols are adapter layers," see: [Design Reflections on Capability Models for AI Agents and Multi-Protocol Generation Architectures](https://tansm.github.io/2026/06/09/Design_Reflections_on_Capability_Models_for_AI_Agents_and_Multi_Protocol_Generation_Architectures.html)
