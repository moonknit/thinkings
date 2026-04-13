# MCP Skepticism: Why Model Context Protocol May Have Missed the Point

> A synthesis of architectural observations on why MCP is likely heading in the wrong direction —
> and where it still has a legitimate role.

---

## 1. The Root Problem: Misreading What LLM + Tools Actually Means

MCP was designed under the assumption that AI agents needed a standardized connector layer to interact with external tools and services. But this assumption overlooks what the `LLM + tools + strong prompt` combination already makes possible.

The real power of LLMs is their ability to compose a small number of primitive tools into arbitrarily complex behaviors. MCP responded to this by adding *more tools* — a fundamentally different, and weaker, approach.

---

## 2. Primitive Tools Should Stay Small — Like Primitive Types

The analogy to programming language design is instructive:

- **Primitive tools** = primitive types (bash, http, file I/O, browser)
- **Skills** = classes (composed behaviors, reusable abstractions)

In language design, the fewer primitive types, the more powerful and predictable the language. The same principle applies to agent tool design.

MCP violated this by turning every integration — AWS, Slack, Jira — into its own "primitive." That's like adding `AWSType` and `SlackType` as built-in language types instead of building them as classes on top of a small set of primitives.

The result: combinatorial explosion of tools, reduced flexibility, and an agent that becomes *less capable* as tool count grows.

> LLM performance tends to degrade as the number of available tools increases. More tools means more ambiguity, not more capability.

---

## 3. Encapsulation in the Wrong Place

MCP encapsulates logic on the **server side**. This made sense in a pre-LLM world where clients were dumb and couldn't handle complexity.

But LLMs flip that assumption entirely. The client is now intelligent. It can handle complex tool composition, interpret ambiguous instructions, and recover from errors — all locally.

MCP was designed with the old mental model:

> "Clients are simple. Put complexity on the server."

The LLM-native mental model is:

> "Clients are intelligent. Give them primitive tools and let them compose."

This is the **thin server, fat client** direction — and it's more natural for the LLM era.

---

## 4. The Governance Argument Doesn't Hold

A common defense of MCP is that it's necessary for enterprise use cases: access control, audit logging, permission management. But this argument falls apart under scrutiny.

These concerns are already handled at the infrastructure level:

- **IAM** handles permissions
- **CloudTrail** handles audit logs
- **VPC/Security Groups** handle access control

If you give an agent access to the AWS CLI with appropriate IAM credentials, you get all of that governance *for free* — without an MCP server in the middle.

> MCP doesn't add governance. It adds another layer between the agent and the infrastructure that already provides governance.

---

## 5. The Better Architecture: Client-Bundled Skills

For cases where raw primitive tools are too slow (e.g., complex HTTP orchestration), the right answer is not an MCP server — it's a **client-bundled skill**.

```
┌─────────────────────────────────────┐
│           Prompt / Intent           │  ← natural language interface
├─────────────────────────────────────┤
│         Skill (with bundled client) │  ← pre-built, optimized logic
├─────────────────────────────────────┤
│    Primitive Tools (bash, http...)  │  ← small, stable, composable
└─────────────────────────────────────┘
```

- The **interface** stays open (prompt-driven, flexible)
- The **implementation** stays closed (optimized client code)
- No external server required, no protocol overhead
- Full encapsulation — like a class with private internals

This is what MCP was *trying* to be, achieved without the protocol layer.

Server-side encapsulation only makes sense when:
- Credentials cannot be exposed to the client
- State must be shared across multiple clients
- Execution is genuinely impossible on the client side

Everything else belongs on the client.

---

## 6. Where MCP Actually Has a Legitimate Role

There is one scenario where MCP is genuinely the right choice: **when users cannot use skills at all** — i.e., when developers are building their own agent from scratch rather than leveraging a general-purpose agent platform.

In this case, the developer controls the full stack and needs a structured way to expose tools to their LLM. MCP provides a standardized interface for that — and in the absence of a skill system, it fills a real gap.

```
Direct agent implementation (custom-built)
    └─ No skill layer available
    └─ MCP is a reasonable tool interface contract
```

The key distinction:

| Context | Right choice |
|---|---|
| Using a general-purpose agent (Claude, GPT, etc.) | Skills + primitive tools |
| Building your own agent from scratch | MCP is viable |

---

## 7. The Long-Term Trajectory: Wrapper Agents over Custom Agents

The direct-agent-build use case for MCP is likely to shrink over time.

As general-purpose agents become more capable, the dominant pattern will shift from:

> "Build your own agent and wire up MCP tools"

to:

> "Wrap a general-purpose agent with a thin layer that defines scope, persona, and skill access"

In other words: **wrapper agents** built on top of powerful general-purpose agents, rather than fully custom agent implementations.

```
Today
  Custom Agent ──── MCP Tools ──── External Services

Near Future
  General-Purpose Agent
        └── Wrapper Agent (scope + skills + persona)
                └── Primitive Tools + Client-Bundled Skills
```

This shift progressively eliminates the need for MCP even in the custom-build scenario. As wrapper agents become the standard pattern, the surface area where MCP is "the only option" continues to shrink.

---

## 8. Why MCP Gained Traction Anyway

MCP's rise reflects a broader pattern in tech: when a new capability appears, the initial tooling often mirrors old paradigms rather than embracing the new one.

MCP emerged from the pre-agentic mindset where:
- Clients were thin
- Integrations required dedicated connectors
- Standardization meant server-side contracts

The AI community recognized the need for better tool integration but reached for a familiar solution (client-server protocol) rather than rethinking the architecture from first principles.

The formation of an MCP steering committee (with Microsoft, OpenAI, Google) may slow the decline, but institutional momentum doesn't resolve the architectural mismatch.

---

## Summary

| Dimension | MCP Approach | LLM-Native Approach |
|---|---|---|
| Tool philosophy | Many specialized tools | Few primitive tools + skills |
| Encapsulation | Server-side | Client-side (bundled in skill) |
| Governance | Protocol layer | Existing infra (IAM, CloudTrail) |
| Complexity | High (servers, auth, transport) | Low (prompt + client code) |
| Flexibility | Fixed at design time | Composed at runtime |
| LLM compatibility | Degrades with scale | Improves with capability |
| Legitimate use case | Custom agent builds (no skill layer) | General-purpose + wrapper agents |

**The core insight**: MCP underestimated the power of `LLM + primitive tools + strong prompts`, and tried to solve problems that this combination already handles better — with less infrastructure, less complexity, and more flexibility.

MCP has a real role today in custom agent builds where a skill layer isn't available. But as general-purpose agents mature and wrapper agents become the dominant pattern, even that foothold is likely to erode.

---

*These observations are architectural in nature and reflect the current trajectory of agentic AI development as of 2025.*
