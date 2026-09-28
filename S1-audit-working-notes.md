# Supplementary Material: Audit Working Notes

**Paper:** When Agents Delegate: Accountability Chains in Multi-Agent AI Systems (Bahidika, 2026)

Per-assessment verbatim quotations, source URLs, and rationale for all twenty-five verdicts reported in Section VI of the article. Each artifact's audit was independently re-verified against the cited sources; two initially paraphrased quotations were corrected to exact wording during verification.

## MCP (Model Context Protocol) specification, modelcontextprotocol.io

**Version audited:** Current revision 2026-07-28 (audited 2026-08-25; earlier revisions 2025-06-18 and 2025-11-25 checked for continuity; security language is materially unchanged)

**Sources:** https://modelcontextprotocol.io/specification/2026-07-28; https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization; https://modelcontextprotocol.io/specification/2026-07-28/architecture; https://modelcontextprotocol.io/specification/2025-06-18/basic/security_best_practices; https://modelcontextprotocol.io/specification/versioning

### C1 verdict: partial

**Evidence:** Authorization page (2026-07-28): "Authorization is **OPTIONAL** for MCP implementations." Within a hop, tokens are verifiable and audience-bound at MUST level: "MCP clients **MUST** implement Resource Indicators for OAuth 2.0 as defined in RFC 8707"; "MCP servers **MUST** validate that access tokens were issued specifically for them as the intended audience"; "MCP servers **MUST NOT** accept or transit any other tokens." Cross-hop expansion is blocked but inheritance is not defined: security_best_practices: "MCP servers **MUST NOT** accept any tokens that were not explicitly issued for the MCP server" (token passthrough forbidden). Scope subsetting is advisory only: "the auth server MAY issue a subset of requested scopes"; "MCP clients **SHOULD** follow the principle of least privilege." (https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization; https://modelcontextprotocol.io/specification/2025-06-18/basic/security_best_practices)

**Rationale:** Authorization travels as a verifiable OAuth token, and downstream parties cannot reuse or expand a given token (audience binding + passthrough ban are MUST). But each hop mints a fresh token from an authorization server ('The access token used at the upstream API is a separate token, issued by the upstream authorization server'), so the delegate's effective scope is NOT mechanically derived from or bounded by the delegator's scope; attenuation across the chain depends on each AS's policy, not the protocol. And the whole mechanism is OPTIONAL and HTTP-transport-only (STDIO 'SHOULD NOT' use it).

### C2 verdict: partial

**Evidence:** Authorization Roles: "An *MCP client* acts as an OAuth 2.1 client, making protected resource requests on behalf of a resource owner"; "The *authorization server* is responsible for interacting with the user (if necessary) and issuing access tokens." Spec index Security section (non-RFC2119 lowercase): "Users must explicitly consent to and understand all data access and operations... Hosts must obtain explicit user consent before invoking any tool." Followed by: "While MCP itself cannot enforce these security principles at the protocol level, implementors **SHOULD**: 1. Build robust consent and authorization flows..." (https://modelcontextprotocol.io/specification/2026-07-28; https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization)

**Rationale:** A human resource owner anchors each individual client-to-server OAuth grant (when authorization is used at all), and consent principles presume a human user at the host. But there is no protocol construct carrying an identified natural person end-to-end: each hop re-anchors to whatever the local AS considers the resource owner, tokens are opaque across hops, and client_credentials flows ('Clients acting on their own behalf') are explicitly contemplated with no human principal. The consent 'must' statements are lowercase and expressly unenforceable at protocol level.

### C3 verdict: absent

**Evidence:** Searched the 2026-07-28 spec index, architecture, authorization, and security best practices pages for any requirement to record delegation events (who delegated, to whom, under what goal/scope). Nothing normative found. Closest text is advisory operational guidance: security_best_practices Scope Minimization: "Log elevation events (scope requested, granted subset) with correlation IDs" (non-normative 'server guidance'); stdio proxy section: proxies "**SHOULD**... Log all `stdio` transport usage for security monitoring." The spec itself acknowledges the gap as a rationale, not a requirement: token passthrough causes "Accountability and Audit Trail Issues... make incident investigation, controls, and auditing more difficult." (https://modelcontextprotocol.io/specification/2025-06-18/basic/security_best_practices)

**Rationale:** No MCP message, field, or requirement records a delegation event or its initiator. Attribution exists only implicitly via OAuth client_id at each AS, which is per-hop and invisible to other parties in the chain.

### C4 verdict: absent

**Evidence:** Searched for MUST-level audit/trace/provenance requirements across hops in the spec index, architecture ('Servers should not be able to read the whole conversation, nor "see into" other servers... Full conversation history stays with the host'), authorization, and security best practices pages. No audit-record requirement of any strength exists at protocol level; the only logging mentions are the advisory items quoted under C3. The architecture's isolation principle actively works against chain reconstruction: each server "receive[s] only necessary contextual information" and "maintains isolation." (https://modelcontextprotocol.io/specification/2026-07-28/architecture)

**Rationale:** Not even SHOULD-level per-node logging is required by the core spec; the delegation tree is not reconstructable from protocol artifacts. Downstream token separation ('a separate token, issued by the upstream authorization server') deliberately severs linkability between hops.

### C5 verdict: absent

**Evidence:** Searched the spec index, architecture, authorization (incl. step-up flow), security best practices, and the Tasks extension description for depth limits, recursion limits, or constraints on the admissible set of downstream delegates. None found. The only limits language concerns auth retries, not delegation: "Retry the original request with the new authorization no more than a few times"; "Clients **SHOULD** implement retry limits." Composability is instead a stated design goal: "Servers should be highly composable... Multiple servers can be combined seamlessly." An MCP server may itself act as an OAuth client to upstream APIs with no protocol bound on chain length. (https://modelcontextprotocol.io/specification/2026-07-28/basic/authorization; https://modelcontextprotocol.io/specification/2026-07-28/architecture)

**Rationale:** No protocol-level bound on delegation depth or diameter; any host/server can extend the chain, limited only by app-level configuration.

**Summary:** MCP 2026-07-28 audit. The spec is explicit that "MCP itself cannot enforce these security principles at the protocol level"; its consent model is per host-tool hop and its authorization model is per client-server pair. C1 partial: within a single hop, authorization is a genuine verifiable artifact with strong MUST-level containment (RFC 8707 audience binding, absolute token-passthrough ban), but authorization is OPTIONAL overall, HTTP-only, and each hop mints an independent token, so there is no mechanical scope inheritance or attenuation across a delegation chain. C2 partial: each OAuth grant anchors to a human resource owner and consent principles presume a user at the host, but identity is re-anchored (or lost, in client_credentials flows) at every agent boundary; no construct carries an identified natural person end-to-end. C3 absent: no requirement to record delegation events; only advisory logging guidance in the non-normative best-practices document. C4 absent: no audit/provenance requirement at any RFC-2119 level; the architecture's server-isolation principle makes the full delegation tree unreconstructable by design. C5 absent: no depth, recursion, or delegate-set limits; composability of servers is a stated design goal. Net: MCP conserves authorization only pairwise via OAuth, and provides no protocol-level delegation accountability, provenance, or bounds, which is consistent with the spec's own framing that these are implementor responsibilities.

## A2A (Agent2Agent) protocol, a2a-protocol.org, Linux Foundation (originally Google)

**Version audited:** Specification 1.0.0 (latest published at a2a-protocol.org/latest/specification/), audited 2026-08-25

**Sources:** https://a2a-protocol.org/latest/specification/; https://a2a-protocol.org/latest/topics/enterprise-ready/; https://github.com/a2aproject/A2A/issues/19

### C1 verdict: partial

**Evidence:** Spec 1.0.0 declares per-hop credential schemes in the AgentCard (Section 4.5: APIKey, HTTP, OAuth2, OpenIdConnect, MutualTLS) and mandates per-hop enforcement: the server "MUST authenticate every incoming request based on the provided credentials and its declared authentication requirements" (Section 7.4) and "Servers MUST implement authorization checks on every A2A Protocol Operations request", with "Implementations MUST scope results to the caller's authorized access boundaries as defined by the agent's authorization model" (Section 7.5, https://a2a-protocol.org/latest/specification/). The spec also requires that "Servers MUST reject requests with invalid or missing authentication credentials" (error-handling requirements), and its one chain-aware construct is scoped narrowly: a credential obtained while a task is in an authorization-required state "MUST NOT be assumed to authorize subsequent messages on the Task unless that behavior is explicitly defined by the implementation, credential issuer, or extension" (Section 7, https://a2a-protocol.org/latest/specification/). Enterprise guidance adds "Agents must grant only the necessary permissions required for a client or user to perform their intended operations" (least privilege, https://a2a-protocol.org/latest/topics/enterprise-ready/). However, the same guidance states "A2A protocol payloads, such as JSON-RPC messages, don't carry user or client identity information directly. Identity is established at the transport/HTTP layer." Searched the spec for delegation, on-behalf-of, scope inheritance, capability propagation: none found.

**Rationale:** Authorization travels only as per-hop transport credentials each agent obtains independently. There is no protocol mechanism forcing a downstream agent's effective scope to be a subset of the delegator's; OAuth2/OIDC scheme declarations are app-implementable hooks (e.g. token-exchange patterns discussed in community material such as GitHub issue a2aproject/A2A#19), not a protocol-level conserved-authorization artifact. Nothing prevents a hop from using its own broader service credentials downstream.

### C2 verdict: partial

**Evidence:** OpenIdConnectSecurityScheme is defined at protocol level (Section 4.5.5, https://a2a-protocol.org/latest/specification/), so a human principal CAN be authenticated at a hop. But "A2A protocol payloads ... don't carry user or client identity information directly. Identity is established at the transport/HTTP layer" (https://a2a-protocol.org/latest/topics/enterprise-ready/). Searched for: end user, principal, human, user identity propagation across agents; no normative end-to-end identity requirement found in the spec.

**Rationale:** A human can be identified at the first client-server boundary via OIDC/OAuth, but the protocol carries no user identity in messages, so identity is lost at the first agent boundary unless implementations layer their own passthrough. No MUST-level requirement that a chain terminate in an identified natural person.

### C3 verdict: absent

**Evidence:** Searched Spec 1.0.0 (https://a2a-protocol.org/latest/specification/) for any requirement to record who delegated, to whom, and under what scope: none exists. The spec defines contextId grouping (Section 3.4.1) and referenceTaskIds (Section 4.1.4), which are informational task references, not delegation records. The only nearby normative text is data-access scoping: "Implementations MUST enforce authorization scoping to ensure clients can only access authorized tasks" (Section 13.1), which is an access control, not an attribution requirement.

**Rationale:** The protocol does not model a delegation event at all; a downstream A2A call is just a new client-server request. No required record of delegator identity, goal, or granted scope.

### C4 verdict: partial

**Evidence:** Audit/tracing appears only in non-normative enterprise guidance at SHOULD level: "A2A Clients and Servers should participate in distributed tracing systems ... propagate trace context ... through standard HTTP headers" and "Audit significant events, such as task creation, critical state changes, and agent actions, especially when involving sensitive data" (https://a2a-protocol.org/latest/topics/enterprise-ready/). The specification itself (https://a2a-protocol.org/latest/specification/) contains no normative audit, logging, or tracing requirements; searched for audit, log, trace, provenance.

**Rationale:** Chain reconstruction is possible only if every implementation voluntarily adopts W3C-style trace propagation; nothing at MUST level guarantees the delegation tree is reconstructable from the origin.

### C5 verdict: absent

**Evidence:** Searched Spec 1.0.0 (https://a2a-protocol.org/latest/specification/) and the enterprise-ready topic page for: depth, recursion, hop limit, chain length, allowed/admissible downstream agents. No verbatim text found. The spec places no limits on nested task chains or on which agents a server may itself call; agent discovery via AgentCards is open-ended.

**Rationale:** Any bound on delegation depth or on the admissible delegate set would be purely application-level configuration; the protocol is silent.

**Summary:** A2A 1.0.0 is a per-hop client-server protocol: it has strong MUST-level transport security and per-request authentication/authorization (TLS, AgentCard securitySchemes incl. OAuth2/OIDC/mTLS, mandatory credential validation and authorization scoping), but it does not model delegation as a first-class concept. Payloads explicitly carry no user or client identity; identity lives at the transport layer and is lost at each agent boundary unless implementations add their own passthrough (C1 partial, C2 partial). There is no requirement to record delegation events (C3 absent), audit and distributed tracing are SHOULD-level guidance in a non-normative enterprise page rather than the spec (C4 partial), and no depth, recursion, or admissible-delegate limits exist anywhere (C5 absent). Net: A2A secures individual hops well but provides no conserved authorization, human anchoring, or chain-complete provenance across multi-agent chains at protocol level.

## LangGraph (LangChain), multi-agent documentation: docs.langchain.com OSS Python docs (multi-agent, graph-api, interrupts) and langchain-ai.github.io/langgraph

**Version audited:** docs.langchain.com OSS Python docs as of 2026-08-25; recursion_limit behavior documented for langgraph >= 1.0.6

**Sources:** https://docs.langchain.com/oss/python/langchain/multi-agent; https://docs.langchain.com/oss/python/langgraph/graph-api; https://docs.langchain.com/oss/python/langgraph/interrupts; https://langchain-ai.github.io/langgraph/concepts/multi_agent/

### C1 verdict: absent

**Evidence:** The multi-agent docs describe handoffs only as control transfer: "Agents transfer control to each other via tool calls. Each agent can hand off to others or respond directly to the user." (https://docs.langchain.com/oss/python/langchain/multi-agent). Searched the multi-agent, graph-api, and interrupts pages for authorization, permission scoping, capability tokens, or scope inheritance; nothing found.

**Rationale:** Handoffs pass graph state and control (Command/goto), not a verifiable authorization artifact. A subagent's tools/scope are whatever the developer wires in; nothing mechanically subsets the delegator's scope, and downstream nodes can be given arbitrary tools. Delegation intent travels as prompts/state (natural language/config).

### C2 verdict: partial

**Evidence:** "Interrupts allow you to pause graph execution at specific points and wait for external input before continuing. This enables human-in-the-loop patterns." and "One of the most common uses of interrupts is to pause before a critical action and ask for approval." But the documentation does not explicitly identify or name specific human principals; it references generic roles like 'caller,' 'reviewer,' 'user,' and 'human' without specifying identity requirements or authentication mechanisms. (https://docs.langchain.com/oss/python/langgraph/interrupts)

**Rationale:** interrupt() is a first-class framework hook an app can use to anchor a human, but it is optional, per-node, and carries no identity of the approving person. No end-to-end human principal is represented in the framework; identity is lost at the first agent boundary unless the app builds it.

### C3 verdict: partial

**Evidence:** Tracing is advisory only: "Trace the full coordination flow across agents with LangSmith", which the docs page presents as a recommendation, not a requirement (https://docs.langchain.com/oss/python/langchain/multi-agent). Searched for any MUST-level requirement that handoff/delegation events be recorded with delegator, delegatee, and goal; none found.

**Rationale:** If a checkpointer is configured (optional), handoffs appear in persisted state history, and LangSmith (optional external service) records agent-to-agent calls. Neither is mandatory, and neither captures 'under what scope' as a typed field.

### C4 verdict: partial

**Evidence:** Only advisory language exists: "Trace the full coordination flow across agents with LangSmith" and "set up LangSmith Engine which monitors your traces, detects issues, and proposes fixes", which is advisory rather than prescriptive language about requirements (https://docs.langchain.com/oss/python/langchain/multi-agent). No MUST-level cross-hop audit requirement found on any page searched.

**Rationale:** When LangSmith tracing or checkpointing is enabled, the full call tree within one graph run is reconstructable, including subgraph invocations. But it is opt-in, and provenance across separate deployments/processes is not addressed by the framework.

### C5 verdict: partial

**Evidence:** "Starting in version 1.0.6, the default recursion limit is set to 1000 steps." "Once the limit is reached, LangGraph will raise GraphRecursionError." "The recursion limit can be set on any graph at runtime, and is passed to invoke/stream via the config dictionary." (https://docs.langchain.com/oss/python/langgraph/graph-api)

**Rationale:** This is a genuinely enforced, always-on framework mechanism (a default limit with a hard error), which is stronger than a pure app hook. However, it bounds total super-steps of a run, not delegation depth or the admissible set of delegates; it is freely raisable by the caller; and the docs do not state how it composes across subgraphs. No limit on who may be delegated to exists; the graph topology is entirely developer-defined.

**Summary:** LangGraph is an orchestration framework, not a delegation-governance protocol, and its docs reflect that. Handoffs (supervisor pattern, Command/goto, subgraphs) transfer control and shared state between agents with no notion of authorization scope, capability tokens, or scope subsetting (C1 absent). Human oversight exists only as the optional interrupt() primitive, which pauses for "external input" but never identifies or authenticates the human principal (C2 partial). Recording of delegation events and cross-hop provenance is available via optional checkpointing and LangSmith tracing, recommended but never required (C3, C4 partial). The one mechanically enforced bound is recursion_limit: a default of 1000 super-steps with a hard GraphRecursionError, but it caps run length, not delegation depth or the set of admissible delegates, and is arbitrarily configurable (C5 partial). Net: strong app-implementable hooks, no MUST-level governance guarantees at framework level.

## AutoGen (Microsoft multi-agent framework, AgentChat/Core, microsoft.github.io/autogen)

**Version audited:** Stable docs at microsoft.github.io/autogen/stable/ corresponding to autogen-agentchat 0.7.x (latest PyPI 0.7.5); audited 2026-08-25

**Sources:** https://microsoft.github.io/autogen/stable/; https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/swarm.html; https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/human-in-the-loop.html; https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/termination.html; https://microsoft.github.io/autogen/stable/user-guide/core-user-guide/framework/telemetry.html; https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/teams.html

### C1 verdict: absent

**Evidence:** Swarm docs describe delegation purely as model tool calls in shared conversational context: "agents can hand off task to other agents based on their capabilities"; "The AssistantAgent uses the tool calling capability of the model to generate handoffs"; "all agents share the same message context" (https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/swarm.html). Searched the framework homepage, swarm/handoff, teams, and human-in-the-loop pages for authorization, capability tokens, scope inheritance, permissions, or verifiable delegation artifacts: none found. Authorization travels only as natural-language messages plus Python-level configuration.

**Rationale:** The `handoffs` parameter restricts which targets an agent may hand off to, but this is app-level constructor config, not a verifiable, non-expandable authorization artifact; there is no scope-subsetting mechanism between delegator and delegate.

### C2 verdict: partial

**Evidence:** Human-in-the-loop docs: UserProxyAgent is "a special built-in agent that acts as a proxy for a user to provide feedback to the team" and "when the team calls the UserProxyAgent, it blocks the execution of the team until the user provides feedback or errors out"; Swarm: "If an agent hands off to the 'user', the team execution will stop and wait for the user to input a response" (https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/human-in-the-loop.html, https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/swarm.html). The docs contain no mechanism identifying or authenticating that human; searched for identity, authentication, and accountable-principal concepts and found none.

**Rationale:** A human control point exists and can block execution, but it is optional (apps need not include a UserProxyAgent) and the human is an anonymous input() callback, not an identified natural person; identity is lost at the first agent boundary.

### C3 verdict: partial

**Evidence:** Swarm docs: "When an agent generates a HandoffMessage, the receiving agent takes over the task with the same message context" and "the speaker agent is selected based on the most recent HandoffMessage message in the context"; the HandoffMessage carries source and target (https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/swarm.html). No requirement found that delegation events be durably recorded or attributed beyond the in-memory message stream; searched for audit/attribution requirements and found none.

**Rationale:** Each handoff is structurally represented with source/target, so attribution is possible within a run, but persistence is not mandated and only Swarm uses HandoffMessage; SelectorGroupChat/RoundRobinGroupChat select speakers without any delegation record beyond ordinary messages.

### C4 verdict: partial

**Evidence:** Telemetry docs: AutoGen has "native support for open telemetry" with runtime, tool, and agent spans following the "GenAI semantic convention", but it is opt-in: users must configure a tracer provider, and can "set the environment variable AUTOGEN_DISABLE_RUNTIME_TRACING to true to disable" tracing (https://microsoft.github.io/autogen/stable/user-guide/core-user-guide/framework/telemetry.html). No MUST-level requirement that traces be produced or that a full delegation tree be reconstructable across hops.

**Rationale:** Chain-complete provenance is achievable via OpenTelemetry instrumentation and shared message history, but it is optional/disable-able tooling, not a normative protocol guarantee, and cross-process (GrpcWorkerAgentRuntime) completeness depends on deployment configuration.

### C5 verdict: partial

**Evidence:** Termination docs: "A run can go on forever, and in many cases, we need to know when to stop them", followed by 11 optional termination conditions (MaxMessageTermination, TimeoutTermination, HandoffTermination, etc.) (https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/termination.html). Teams docs: max_turns is "The maximum number of turns in the group chat before stopping" and "defaults to None, meaning no limit" (https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/tutorial/teams.html). The Swarm `handoffs` parameter bounds admissible delegates per agent. Searched for mandatory recursion or depth limits: none found.

**Rationale:** Bounds on turns/messages/tokens/time and a per-agent admissible-delegate list exist, but every one is app-level configuration with unlimited defaults; there is no protocol-level recursion or depth limit.

**Summary:** AutoGen (stable docs, autogen-agentchat 0.7.x) is a programming framework, not a governance protocol, and its docs make no normative (MUST-level) guarantees for any of the five conditions. C1 is absent: handoffs are model tool calls in a shared message context with no capability/token artifact and no scope inheritance; the per-agent `handoffs` list is only constructor config. C2 is partial: UserProxyAgent and handoff-to-'user' give a blocking human control point, but the human is an anonymous input callback with no identification or authentication, and including one is optional. C3 is partial: Swarm's HandoffMessage structurally records source and target of each delegation within the run, but recording is not required, not durable by default, and other team types have no delegation record. C4 is partial: native OpenTelemetry instrumentation (runtime, tool, agent spans per GenAI semantic conventions) can reconstruct chains, but it is opt-in and explicitly disable-able (AUTOGEN_DISABLE_RUNTIME_TRACING). C5 is partial: max_turns and 11 termination conditions can bound runs and `handoffs` bounds admissible delegates, but max_turns "defaults to None, meaning no limit" and no protocol-level depth/recursion limit exists. Fairness note: AutoGen never claims to be an interoperability/authorization protocol; scoring it against protocol-grade MUST requirements necessarily yields partial/absent, and all five conditions are implementable on top of its hooks.

## OpenAI-Agents-SDK

**Version audited:** openai.github.io/openai-agents-python living docs, unversioned, fetched 2026-08-25 (pages: handoffs, tracing, running_agents, guardrails, human_in_the_loop, config)

**Sources:** https://openai.github.io/openai-agents-python/handoffs/; https://openai.github.io/openai-agents-python/tracing/; https://openai.github.io/openai-agents-python/running_agents/; https://openai.github.io/openai-agents-python/guardrails/; https://openai.github.io/openai-agents-python/human_in_the_loop/; https://openai.github.io/openai-agents-python/config/

### C1 verdict: absent

**Evidence:** Handoffs doc: "When a handoff occurs, it's as though the new agent takes over the conversation, and gets to see the entire previous conversation history." and "If you want to change this, you can set an `input_filter`. An input filter is a function that receives the existing input via a `HandoffInputData`, and must return a new `HandoffInputData`." (https://openai.github.io/openai-agents-python/handoffs/). Searched handoffs, guardrails, and config pages for scope/permission inheritance, capability tokens, or authorization artifacts constraining the receiving agent; none found.

**Rationale:** What travels at a handoff is conversation history (natural-language context), not authorization. Each Agent's tools are independently developer-configured; nothing makes the delegate's effective capability a subset of the delegator's, and the delegate can be defined with strictly broader tools. input_filter is an app-implementable hook over history, not over authority, so it does not qualify even as partial scope conservation. Guardrails explicitly do not follow the chain: "Input guardrails run only for the first agent in the chain."

### C2 verdict: absent

**Evidence:** Human-in-the-loop doc: approval is opt-in via `needs_approval=True`; "`RunResult.interruptions` contains `ToolApprovalItem` entries with details such as `agent.name`, `tool_name`, and `arguments`." (https://openai.github.io/openai-agents-python/human_in_the_loop/). Searched human_in_the_loop, tracing, and config pages for user identity, accountable principal, or attribution of approvals to a named person; none found. Config doc only ties traces to an API key: "By default it uses the same OpenAI API key as your model requests."

**Rationale:** Approvals are decisions in a RunState, not actions attributed to an identified natural person. Identity resolution stops at the API key / developer org; no end-to-end human principal exists at framework level.

### C3 verdict: partial

**Evidence:** Tracing doc: "Tracing is enabled by default. You can disable it in three common ways"; "Handoffs are wrapped in `handoff_span()`"; "Each time an agent runs, it is wrapped in `agent_span()`"; also "You can globally disable tracing by setting the env var `OPENAI_AGENTS_DISABLE_TRACING=1`" and "Tracing is unavailable for organizations that use OpenAI's APIs under a Zero Data Retention (ZDR) policy." (https://openai.github.io/openai-agents-python/tracing/)

**Rationale:** Delegation events (handoffs) are attributable by default via handoff_span/agent_span, capturing which agent handed to which within a run. But recording is opt-out, disable-able by env var, and structurally unavailable under ZDR, so it is not a MUST-level requirement. Goal/scope of the delegation is not a first-class recorded field.

### C4 verdict: partial

**Evidence:** Tracing doc: "The entire `Runner.{run, run_sync, run_streamed}()` is wrapped in a `trace()`... Handoffs are wrapped in `handoff_span()`"; the span tree covers all hops of a run, so the delegation tree is reconstructable when tracing is on. But: "You can also disable tracing entirely by using the `set_tracing_disabled()` function." (config page) and "set `RunConfig.trace_include_sensitive_data` to `False`" strips payloads (https://openai.github.io/openai-agents-python/tracing/, /config/). Handoffs doc: "Handoffs stay within a single run."

**Rationale:** Chain-complete within a single Runner invocation and on by default, which is stronger than most SDKs, but optional (opt-out), payload-redactable, and absent under ZDR. Provenance across separate runs or across processes/organizations is not guaranteed. Fails the MUST test; clear partial.

### C5 verdict: partial

**Evidence:** Running-agents doc: "If we exceed the `max_turns` passed, we raise a `MaxTurnsExceeded` exception."; "Pass `max_turns=None` to disable this turn limit."; "If the LLM requests a handoff, we update the current agent and input, and re-run the loop" (https://openai.github.io/openai-agents-python/running_agents/). Searched for explicit delegation-depth limits or restrictions on the admissible delegate set beyond the developer-declared handoffs list; none found.

**Rationale:** max_turns bounds total loop iterations per run (handoffs re-enter the loop, so they are indirectly bounded), and the delegate set is closed to the statically declared handoffs list per agent, which bounds the diameter graph. But max_turns is per-call configuration, disable-able with None, and there is no protocol-level recursion or depth limit distinct from app config.

**Summary:** The OpenAI Agents SDK is a single-process orchestration framework, not an authorization protocol. Handoffs transfer conversation history, not scoped authority: the receiving agent's capabilities are whatever the developer statically configured, with no mechanical subset relation to the delegator and no verifiable authorization artifact (C1 absent). No accountable human principal exists; approvals and traces attach to agent names and API keys, not identified persons (C2 absent). Its strongest property is tracing: on by default and chain-complete within a run (agent_span/handoff_span per hop), making the delegation tree reconstructable, but it is opt-out (env var, set_tracing_disabled), payload-redactable, and unavailable under ZDR, so C3/C4 score partial, not satisfied. Depth is bounded only by the per-run max_turns config (disable-able via None) and the statically declared handoff lists; there is no protocol-level recursion limit (C5 partial). Docs are unversioned living pages; audited as fetched 2026-08-25. Fairness note: within-run default-on tracing exceeds SHOULD-level baselines common elsewhere, and the closed handoff list is a real (if configurational) diameter bound.
