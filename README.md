# The Integrity Burden: Why Agentic Document Editing Belongs in the Infrastructure, Not the Agent

**John Majerus**  
*Draft — September 2026*

---

## Abstract

Current agentic frameworks treat document editing as a round-trip operation: an agent receives a complete document, makes targeted changes, and returns it intact. This model places the burden of document integrity on the agent — a burden it is structurally ill-suited to discharge. The result is a proliferation of workarounds: sidecar files, phased schemas, partial APIs, read-only field conventions. These patches share a common flaw: they attempt to approximate a capability the infrastructure does not provide, at increasing cost to schema designers, authors, and the agents themselves. We argue that the missing primitive is *write-domain scoping* — a formal, infrastructure-enforced declaration of what an agent owns within a document, rendering everything outside that domain invisible to the agent and immutable by construction. This is not a new concept — the object-capability tradition has formalized least authority and ambient authority for decades — but it is a missing *instantiation*: the principle has never been applied as a default infrastructure primitive for the document editing context. The difference this instantiation makes is the difference between a document processor, whose responsibility is faithful round-tripping, and a scoped editor, whose responsibility ends at its declared domain. We make the case that until this primitive exists at the infrastructure level, agentic document editing will continue to solve the wrong problem cleverly rather than the right problem simply.

---

## 1. Introduction

The dominant model for agentic document editing is deceptively simple: hand the agent a document, ask it to make a change, receive the document back. This round-trip model maps naturally onto the way language models work — they consume context and produce completions — and it has the surface appeal of generality. The agent doesn't need to know anything about the document's structure in advance; it just reads, edits, and returns.

The problems with this model may not be immediately obvious, because in simple cases it works tolerably well. Ask an agent to rewrite a paragraph, fix a grammatical error, or update a date, and the round-trip model is adequate. The document is short, the change is small, the risk of corrupting something untouched is low.

But structured documents — documents with complex schemas, multiple sections serving different purposes, fields maintained by different systems or stakeholders — expose the model's fundamental inadequacy. In these cases, the agent is asked to do two things simultaneously: make a targeted, intelligent change to one part of the document, and faithfully reproduce everything else exactly as it was. The first task is what the agent is for. The second task is an integrity constraint that the agent has no reliable mechanism to satisfy.

As complexity increases, the consequences are not merely quality degradation. In observed authoring workflows, the integrity burden has proved the difference between a successful session and a failed one — cases where the agent's capacity to do the actual work was exhausted by the overhead of reproducing what it never should have touched. Laban, Schnabel & Neville's DELEGATE-52 benchmark quantifies the pattern: even frontier models corrupt approximately 25% of document content in long workflows, with average degradation reaching 50% across all tested models after 20 round-trip interactions.

This paper argues that the integrity burden — the requirement that an agent reproduce faithfully what it did not change — is not a solvable engineering problem within the round-trip model. It is a symptom of a category error: treating agents as document processors when what is needed is a different primitive entirely. We call that primitive *write-domain scoping*, and we argue that it must exist at the infrastructure level, not be approximated through schema conventions or application-layer workarounds.

---

## 2. The Integrity Burden, Illustrated

To make the problem concrete, consider a structured authoring environment for educational content — a system where puzzle documents contain clusters of terms, bridge relationships, pedagogical lenses, learning introductions, provenance metadata, and publication fields. The document schema is rich: dozens of optional fields, some player-facing, some system-generated, some serving internal bookkeeping, some recording authorship and AI contribution.

An agent working in this environment is asked to draft the core educational content: term clusters, bridge facts, lens prompts. That is its domain — the part of the document that requires knowledge, judgment, and creativity. But the round-trip model hands the agent the entire document. To save its changes, it must return the entire document. Every field it did not touch must be reproduced exactly.

The failure modes are numerous and well-documented in practice:

**Hallucinated metadata.** Fields that record cross-references to other documents — shared concept identifiers, related puzzle links annotated with shared vocabulary — get populated with plausible-sounding values that don't correspond to anything real. The agent invents coherent-looking data because the field exists and expects a value. Validation catches some of these; others sit in the corpus looking authoritative.

**Stale configuration values.** Fields recording system-specific settings — model configuration parameters, schema version identifiers, processing flags — get reproduced from the agent's context rather than updated to reflect current state. The agent has no reliable way to know whether what it read is still current.

**Round-trip corruption.** Subtle schema conventions — field ordering, null vs. absent, string vs. object representations of the same logical value — get normalized by the agent in ways that are semantically equivalent but structurally different from what the system expects. Validation flags these as errors; the agent fixes them; they reappear.

**Misclassification of constrained fields.** Fields with enumerated values and complex classification rules — relationship taxonomies, contribution role vocabularies — get assigned plausible but incorrect values. The agent applies its best judgment to a classification task it wasn't designed to perform reliably.

In each case, the agent's error is not in the domain it was asked to work in. The errors are all in fields outside its domain — fields it had to read, reproduce, and in some cases populate, not because those fields needed its attention but because the round-trip model gave it no other option.

The response to these failure modes, in practice, is to design around them. Sidecar files carry system-generated metadata separately from authored content. Phased schemas present only a subset of fields to the agent at each stage. Partial APIs send the agent only what it needs, merging on the server side. Read-only field conventions signal — without enforcing — that certain fields should be left alone. Validation rules catch the most common errors after the fact.

These are all workarounds. They reduce the frequency and severity of integrity failures without eliminating their root cause. And they accumulate: each workaround adds schema complexity, authoring conventions, agent instructions, and validation rules that must be learned, maintained, and occasionally debugged. The complexity budget is spent managing the integrity burden rather than on the actual work.

---

## 3. Why Workarounds Cannot Solve the Problem

The workarounds described above share a structural property: they push responsibility for document integrity somewhere other than the infrastructure. Either the agent is expected to respect conventions it has no mechanism to enforce on itself, or the schema designer is expected to anticipate and constrain every way an agent might corrupt a field it shouldn't touch, or the author is expected to review every output carefully enough to catch subtle metadata errors alongside substantive content judgments.

None of these is the right place for the responsibility.

**Agents cannot reliably enforce conventions on themselves.** An agent can be instructed not to modify certain fields. It will generally comply. But compliance is probabilistic, not guaranteed, and it degrades with context length, task complexity, and the number of conventions being simultaneously maintained. An agent holding a dozen "do not touch" rules while also performing substantive creative work is not in a reliable state. The integrity guarantee is only as strong as the agent's attention in that moment.

**Schema designers cannot anticipate every failure mode.** The enumeration of workarounds is necessarily incomplete. A new agent with different tendencies, a schema revision that introduces a new field type, a task that shifts the agent's attention in unexpected ways — any of these can produce a failure mode the schema was not designed to catch. Defensive schema design is an arms race against an adversary that doesn't have fixed behaviors.

**Authors cannot substitute for infrastructure.** Careful human review catches errors that automated validation misses. But human review is expensive, cognitively demanding, and inconsistent. Requiring authors to verify that an agent correctly reproduced every field it wasn't supposed to touch is a tax on human attention that grows with document complexity. It is also a category error: the author's judgment should be applied to the quality of the content, not to the mechanical integrity of fields outside the agent's domain.

The deeper point is that workarounds solve a symptoms while leaving the cause intact. The cause is the round-trip model itself — the requirement that agents bear responsibility for the integrity of content they did not author and cannot reliably assess. No amount of clever schema design eliminates that requirement; it only makes compliance marginally more tractable.

---

## 4. The Missing Primitive: Write-Domain Scoping

What the round-trip model lacks is a formal notion of *what the agent owns*. In the current model, ownership is implicit and total: the agent receives the document, so it is responsible for all of it. Write-domain scoping makes ownership explicit and partial: the agent is declared the owner of specific fields or sections, and the infrastructure enforces that declaration.

The primitive has three components:

**Declaration.** A document schema includes explicit write-domain annotations — markers on fields, sections, or subtrees indicating which agent role or capability is authorized to modify them. These annotations are part of the schema contract, not a separate convention layer.

**Enforcement.** When an agent is handed a document for editing, the infrastructure presents only its declared domain. Fields outside the domain are not transmitted to the agent. They are not in its context. It cannot read them, reproduce them incorrectly, or accidentally modify them. The integrity of out-of-domain fields is guaranteed by the infrastructure, not by the agent's compliance with instructions.

**Merge.** When the agent returns its changes, the infrastructure merges them back into the full document. Out-of-domain fields, maintained separately, are recombined with the agent's contributions. The agent never sees this operation; it is not part of its task.

This is not a partial API — a performance optimization that sends less data to reduce token consumption. It is a capability boundary. The agent's write domain is a formal declaration of its authority within the document, enforced at the system level. An agent that attempts to modify an out-of-domain field — through hallucination, misclassification, or accidental round-trip normalization — finds that its modification has no effect, because the field was never in its possession.

The distinction matters because partial APIs still leave agents responsible for the integrity of what they receive. If the API sends a subset of fields, the agent must reproduce that subset faithfully. The integrity burden is reduced in scope but not eliminated in kind. Write-domain scoping eliminates it: the agent is responsible only for producing good content within its domain, not for reproducing anything else.

---

## 5. What Changes

Write-domain scoping changes the design space for agentic document systems in ways that go beyond error reduction.

**Schema design becomes compositional.** Instead of a single flat schema that every agent must navigate carefully, a schema can be designed as a composition of domains — each with its own owner, its own validation rules, its own lifecycle. The educational content domain has different owners, different constraints, and different change frequencies than the provenance domain or the publication domain. Designing them separately, and composing them at the infrastructure level, produces schemas that are simpler in each domain and more coherent overall.

**Agent tasks become well-defined.** An agent with a declared write domain has a precise task: produce good content within that domain. It does not need instructions about what not to touch, conventions about field ordering, or warnings about classification pitfalls in adjacent fields. Its success criterion is the quality of its domain's content, full stop.

**Multiple agents become composable.** A document that requires contributions from agents with different capabilities — one for core content, one for pedagogical annotation, one for provenance recording, one for publication metadata — can be assembled from those contributions without any single agent being responsible for the whole. Each agent works in its domain; the infrastructure composes the result. This is not multi-agent orchestration in the current sense, which still requires one agent to hand off a full document to another. It is parallel composition with infrastructure-enforced isolation.

**Human review becomes meaningful.** When agents are responsible only for their domains, human review can focus on what humans are actually good at: judging the quality, accuracy, and appropriateness of content. The mechanical integrity of out-of-domain fields is not a review concern because it is not a failure mode. Human attention is spent where it produces value.

**System-generated fields are genuinely separate.** Fields that record system state — generation timestamps, processing flags, configuration parameters — can live in their own infrastructure-maintained domain, never touching the agent's context. The question of whether an agent should record its own model configuration, or whether a sidecar file should carry that metadata, dissolves: system-generated fields are owned by the system, and agents never encounter them.

---

## 6. The Deeper Argument

The case for write-domain scoping is not only that it reduces errors, though it does. It is that it correctly locates responsibility.

The round-trip model assigns responsibility incorrectly: it makes agents responsible for the integrity of content they did not author, cannot reliably assess, and have no structural reason to understand. This is not a design choice — it is a default that emerged from the way language models work and the way document APIs were designed before agents were a primary use case. The round-trip model is the path of least resistance, not the result of deliberate reasoning about where integrity responsibility belongs.

Write-domain scoping is the result of that reasoning. Integrity of a field belongs with whoever owns that field. System-generated fields are owned by the system. Provenance fields are owned by the provenance infrastructure. Educational content fields are owned by the content agent. Each domain's integrity is maintained by its owner, enforced by the infrastructure, and invisible to everyone else.

This is not a novel principle. It is how software systems manage shared mutable state: through ownership, encapsulation, and access control. The novelty is applying it to agentic document editing — a domain that has, so far, treated agents as powerful but undifferentiated document processors rather than as participants in a structured system with defined responsibilities.

The workarounds exist because the principle has not yet been applied. Sidecar files are manual ownership separation without infrastructure enforcement. Phased schemas are manual domain scoping without formal declaration. Partial APIs are manual context reduction without capability boundaries. Each workaround gestures toward write-domain scoping without implementing it. None eliminates the integrity burden; they only redistribute it.

The elegant path forward is not more workarounds. It is infrastructure that makes the right thing the default: agents own their domains, the system owns everything else, and the boundary is enforced rather than negotiated.

---

## 7. Related Work

Write-domain scoping is not a new concept — it is an application of well-established principles to a context where they have not yet been formalized as a default infrastructure primitive. This section situates the paper's contribution within four bodies of prior work and identifies the precise gap that remains in each.

### Object-capability security and the principle of least authority

The foundational theoretical home for write-domain scoping is the object-capability (ocap) tradition. Mark Miller's unified theory of capability-based access control (Miller, *Robust Composition*, Johns Hopkins, 2006), Miller, Yee & Shapiro's *Capability Myths Demolished* (2003), and Hardy's *The Confused Deputy* (1988) together establish the key vocabulary this paper adopts: *ambient authority* (authority exercised implicitly by virtue of identity or position rather than explicit grant), the *confused deputy* (a trusted agent that misuses authority it holds on behalf of one principal to act on behalf of another), *attenuation* (restricting a capability before delegating it), and the *principle of least authority* (POLA — an agent should hold only the authority it needs for its immediate task, and no more).

The integrity burden, in this vocabulary, is precisely an ambient-authority problem. The round-trip model grants the agent ambient authority over the entire document by handing it the whole thing. Write-domain scoping is the application of POLA to the document editing context: give the agent a capability scoped to its declared domain, and render everything else undesignatable and therefore unmodifiable.

The gap is equally precise: classical ocap operates over objects, files, and language-level references in audited code. It has never been instantiated for *sub-document regions of a structured document* as the unit of capability, nor connected to the LLM-agent invocation model where the "program" issuing writes is a stochastic natural-language process rather than code whose authority can be statically analyzed. This paper's contribution is that instantiation.

### Agentic AI systems: capability gating and runtime enforcement

The most directly relevant body of recent work applies capability and information-flow concepts to LLM agents, primarily to address prompt injection and tool-call authority.

Laban, Schnabel & Neville's "LLMs Corrupt Your Documents When You Delegate" (Microsoft Research, arXiv:2604.15597, April 2026) provides the strongest empirical motivation for this paper's central claim. Their DELEGATE-52 benchmark — 310 environments across 52 professional domains — finds that even frontier models corrupt approximately 25% of document content by the end of long workflows, with an average degradation of roughly 50% across all 19 tested models after 20 round-trip interactions. Their "round-trip relay" simulation directly validates the round-trip pathology this paper diagnoses.

Debenedetti et al.'s CaMeL (*Defeating Prompt Injections by Design*, arXiv:2503.18813, 2025) enforces capability metadata on every value flowing through an agent, separating control and data channels via a custom interpreter that operates outside the LLM. CaMeL establishes the key architectural pattern this paper argues for — security enforced at the infrastructure layer, not through the model's judgment — though it targets tool-call data exfiltration rather than intra-document write scope.

Mellafe Zuvic's *Capability Gates Are Not Authorization* (arXiv:2606.28679) audits LangChain, LlamaIndex, and related frameworks, finding that all provide capability gating by default but none provides a deterministic, fail-closed per-call value authorization gate. This confirms that the authority boundary the paper calls for does not exist as a default in current mainstream frameworks.

**The nearest neighbor: PatchOptic.** Bai & Cai's PatchOptic (arXiv:2607.05483, 2026) independently converges on the core mechanism of write-domain scoping and must be engaged directly. PatchOptic defines a step interface for shared-state LLM workflows in which each step declares a *projected-read view* (what the agent sees), an *authorized write region* (what it may modify), and a *patch-source region* (what it may read when constructing its patch). A runtime verifier checks each proposed JSON Patch against the full state before commit, enforcing the declared contract without trusting the agent to respect it.

PatchOptic's own conclusion captures the shared argument precisely: "Instead of asking the model to carry global security and consistency obligations, the runtime makes those obligations an explicit contract."

Three distinctions remain between PatchOptic and the primitive this paper argues for. First, PatchOptic frames itself as a *workflow step interface* — a contract between a step declaration and a runtime verifier in a shared-state workflow system. This paper argues for write-domain scoping as a *general infrastructure primitive for document editing systems*, a different level of abstraction and a different claim about where the responsibility belongs by default. Second, PatchOptic explicitly acknowledges that it "trusts the policy author" — a wrongly declared write region is enforced as declared. The primitive this paper calls for is one that is default and correct without requiring a policy author to configure it correctly for each workflow step. Third, PatchOptic's future work explicitly identifies deployment in "database updates, file edits, tool calls, and domain-specific operations" as a direction it has not yet addressed — the generalization this paper argues is the necessary destination.

### Collaborative document editing and field-level access control

The collaborative editing literature has formalized field- and section-level write authority for human multi-writer systems. Imine, Cherif & Rusinowitch's work on access control for distributed collaborative editors (SDM 2009; Pervasive & Mobile Computing, 2014) applies optimistic, replicated authorization policies with retroactive enforcement. Damiani et al.'s fine-grained XML access control (ACM TISSEC, 2002) enforces access restrictions directly on document structure via security annotations. CRDT-based access control approaches extend these mechanisms to eventually-consistent distributed systems.

This literature establishes that field/section-level write authority is a tractable engineering problem with known solutions — the structural equivalent of write-domain scoping exists here. The gap is that it was built entirely for *human multi-writer concurrency*: the goals are convergence, intention preservation, and consistency among mutually trusted editors. No collaborative editing work treats an AI agent as a principal whose write authority must be scoped, nor addresses the *invisibility* property — that unscoped content should be absent from the agent's context entirely, not merely protected from modification.

A sharper observation: agentic frameworks that rely on whole-document round-trips have regressed below the collaborative-editing state of the art. OT and CRDT systems operate on *operations*, not whole-document replacements. The round-trip LLM interaction — whole doc in, whole doc out — has no analog in collaborative editing systems precisely because the editing literature recognized decades ago that whole-document replacement is not a safe primitive for concurrent authorship.

### Provenance, attribution, and session capture

The C2PA Technical Specification (v2.1–2.4, 2024–2026) provides the closest standards-track mechanism for attributing document edits to specific agents and regions. C2PA's "Regions of Interest" (introduced in spec 1.3, April 2023; extended to text-based formats in v2.3) allows per-action attribution with a `softwareAgent` field and a `changes` array of region maps. The `c2pa.ai-disclosure` assertion records model provenance and human-oversight degree.

PROV-AGENT (Souza et al., IEEE e-Science 2025, arXiv:2508.02866) extends W3C PROV with an `AIModelInvocation` entity that models agent identity, tool use, and response generation, implemented via cooperative in-process instrumentation in the Flowcept framework.

Both approaches are meaningful advances. Both fall short of the companion primitive this paper identifies. C2PA's cryptographic binding operates at whole-claim granularity via a single signer and is self-asserted — a valid signature certifies the metadata was not modified since signing, but does not certify the semantic truth of the assertions or that the attribution was captured externally at the moment of invocation. PROV-AGENT models the invocation boundary but via cooperative in-process instrumentation with no tamper-evidence, recording only agent identity and name in its current implementation.

The unoccupied position is the intersection of four properties simultaneously: *region/field-level* attribution, *invocation-boundary* capture, *external* (not self-asserted) recording, and *tamper-evident* integrity. No existing standard or research artifact occupies this intersection. The IETF SCITT architecture (Signed Claims → Transparency Service → Merkle-anchored receipts) provides the closest standards-track mechanism for the external-capture and tamper-evidence properties, but has not been connected to per-region document edit attribution. Closing this gap is the task the companion primitive — session provenance capture — is designed to address.

### Summary

The principle behind write-domain scoping is well-established in the object-capability tradition; the mechanism has been independently instantiated by PatchOptic for shared-state workflow steps; field-level access control has been formalized for human collaborative editors; and region-level provenance attribution is advancing in C2PA and PROV-AGENT. The gap this paper addresses is the absence of write-domain scoping as a *default, infrastructure-level primitive for document editing systems generally* — not an opt-in workflow contract, not a human-concurrency mechanism, not a self-asserted provenance assertion, but an enforced boundary between what an agent owns and what it does not, present by default in any system that hands an agent a structured document to edit.

---

## 8. A Counter-Argument: Database Partitioning

Before turning to implementation, a practical objection deserves direct treatment: write-domain scoping already exists, in a form, at the storage layer. Split a document across multiple database columns — `content`, `metadata`, `provenance`, `system` — and each column is trivially write-scoped by the database's own access control. No new primitive is needed; just good data modeling.

This counter-argument is genuinely true and genuinely insufficient, for three reasons that are worth stating precisely.

**It solves the storage problem, not the agent problem.** The integrity burden occurs at the moment the agent is handed a document to work with — before any storage operation. If the agent receives a merged, complete document, as most APIs present it, the database partitioning is invisible to it. The round-trip problem reasserts itself at the API layer regardless of how the underlying data is partitioned. The columns are write-scoped; the agent's context is not.

**It cannot express fine-grained domain boundaries.** Column-level partitioning works when domain boundaries are known, stable, and coarse-grained enough to map cleanly to columns. Real document schemas have boundaries that are finer and more context-dependent than that. A single structured object might contain agent-owned fields alongside system-owned fields — educational content alongside an auto-assigned identifier or a system-derived rendering flag. Expressing that granularity through column partitioning either explodes the schema into dozens of narrow columns or nests JSON within JSON, which recreates the original problem inside a column. The approach scales to coarse decomposition; it does not generalize to the field-level scoping that complex documents require.

**It is a specific implementation of the primitive, not an alternative to it.** Multi-column JSON storage is one way to realize write-domain scoping at the persistence layer. The primitive itself is the concept — that domain ownership should be formally declared and infrastructure-enforced. The database pattern validates the principle while demonstrating its limits: it works where the decomposition is natural and coarse, and breaks down where it isn't. Elevating the primitive to a first-class concept in the agentic document editing layer allows it to be applied at the granularity and abstraction level where the integrity burden is actually felt, rather than approximated at the storage level where the decomposition happens to be convenient.

The counter-argument thus strengthens rather than undermines the paper's position. If database partitioning is already a recognized pattern for managing write ownership across systems, that is evidence the underlying principle is sound and well-understood. The gap is not in the principle but in its expression: it has not been formalized as a first-class primitive in the layer where agents operate, leaving practitioners to reach for storage-layer approximations when the problem surfaces at the API layer.

---

## 9. A Proposed Implementation: Database Partitioning in Practice

*Note: The following describes a proposed implementation in the Concept Clusters authoring system \[CITATION: jmajerus/concept-clusters, docs/STORAGE-DOMAINS.md — document to be created\]. It will be revised to reflect the working implementation once complete.*

The counter-argument of Section 7 — that database column partitioning approximates write-domain scoping at the storage layer — points toward a practical path for systems that cannot wait for a first-class infrastructure primitive. This section describes how that approach is being applied in the Concept Clusters authoring system \[CITATION: jmajerus/concept-clusters\], what it achieves, and where its limits surface in practice.

### The Domain Decomposition

The Concept Clusters puzzle document, as evolved through agent-assisted authoring, naturally decomposes into four distinct ownership domains:

**Content** — the educational core that agents are hired to produce: cluster names, term lists, seed pairs, bridge facts, lens prompts and explanations, learning introduction prose. This is what requires knowledge, judgment, and creativity. It is the agent's domain.

**Pedagogy** — structural annotations layered on top of content: lens sequences, bridge relationship kinds, bridge directions, ideal terms. These require authoring judgment but are more mechanical than content and are frequently the source of misclassification errors when agents handle them alongside richer content tasks. Separable from content but still agent-authored.

**Provenance** — who contributed to this puzzle, in what order, with what tools. Currently a flat ordered array of contributor names. The first entry is the drafter by convention; subsequent entries are collaborators. This domain is human-controlled: the author initiates the session, knows the agent and its configuration, and records that information directly. Agents do not write to this domain.

**System** — fields the infrastructure owns entirely: auto-assigned identifiers, derived rendering flags, schema version markers, publication state, timestamps, validation stamps. These were the primary source of agent hallucination and round-trip corruption in the pre-partitioning schema. Agents never see them.

### The Storage Model

Each domain maps to a separate JSON column in the D1 database backing the authoring worker \[CITATION: jmajerus/concept-clusters, docs/STORAGE-DOMAINS.md\]:

```
puzzle_drafts
  content      JSON   -- agent write domain
  pedagogy     JSON   -- agent write domain (separate pass)
  provenance   JSON   -- human write domain
  system       JSON   -- infrastructure write domain
```

The authoring API presents each domain separately. When an agent is asked to draft educational content, it receives only the `content` column. When it is asked to annotate bridge relationships, it receives only the `pedagogy` column — with the content visible as read-only context but not as part of its write surface. The merge that produces a complete document for export or publication is performed by the infrastructure, not by the agent.

### What This Achieves

The most immediate benefit is the elimination of hallucinated metadata. Fields that previously generated plausible-but-wrong values — cross-puzzle concept identifiers, relationship taxonomy classifications, contributor role annotations — are either removed from the schema entirely (as this audit has proposed) or moved to domains the agent never writes. An agent drafting cluster content cannot hallucinate a bridge `relationKind` because `relationKind` is not in its write surface.

The provenance domain's separation is particularly clean. Because the author controls session initiation and knows the contributing agent and its configuration, provenance is recorded once at session start, never touched by the agent, and available for review throughout the authoring process as a visible byline. The agent's contribution to provenance is its output, not its self-report.

System fields — previously a source of round-trip corruption as agents reproduced auto-assigned identifiers in subtly wrong forms — are entirely invisible to agents. The canonicalization logic that previously had to run on every agent save, correcting structural normalizations the agent introduced, is reduced to a thin merge step combining clean domain outputs.

### Where the Limits Surface

The decomposition works cleanly at the domain level. It does not work at the field level within a domain.

Consider the cluster object. In the content domain, a cluster has a `name`, a `fact`, a set of `seeds`, and a set of `floatingTerms`. In the current schema, it also has a `color` — proposed for removal in this audit, derivable from array position. Before that removal, `color` was an agent-written field in the content domain that was functionally a system field: its value was constrained to an enumeration, its assignment was deterministic given cluster order, and agents frequently assigned it incorrectly or inconsistently. It was a system field wearing content clothing.

The database partitioning could not express that boundary. `color` lived in the same JSON column as `name` and `fact`, and the agent wrote all three together. The solution was schema simplification — removing `color` as an authored field — rather than finer-grained partitioning. This is the limit the counter-argument identified: column partitioning scales to coarse domain boundaries, not to field-level ownership within a domain.

The same pattern appeared with `termRole` on bridges, `via` on related puzzle entries, and `conceptId` on bridges — all fields that were in the agent's write surface but were not genuinely the agent's domain. Each required either removal or schema convention to manage, because the partitioning could not enforce the boundary at that granularity.

### The Residual Integrity Burden

What remains after partitioning is not zero. The agent writing to the content domain still bears responsibility for the internal integrity of that domain: term references must be consistent, seed terms must appear in their cluster's term list, bridge cluster references must point to real clusters. These are content-domain integrity constraints, and they are the right kind: they are checkable by validation, they are within the agent's competence to satisfy, and they are directly related to the quality of the educational content being produced.

What is eliminated is the category of integrity burden that has nothing to do with content quality: reproducing system identifiers correctly, classifying relationship types accurately, populating cross-document references without hallucinating values. Those burdens have been moved to domains owned by parties competent to discharge them.

The residual burden is proportionate to the agent's actual domain. The eliminated burden was disproportionate overhead imposed by the round-trip model. That distinction — between integrity constraints that belong to an agent's domain and those that don't — is precisely what write-domain scoping as a first-class primitive would make explicit and enforceable at a finer grain than any column partitioning scheme can achieve.

---

## 10. A Companion Primitive: Session Provenance Capture

*Note: The Concept Clusters workaround described in this section is proposed. It will be revised to reflect the working implementation once complete.*

The domain decomposition of Section 8 exposes a gap that write-domain scoping alone does not close. The provenance domain — who contributed to a document, under what conditions — is described there as human-controlled. That is not the intended endpoint. It is a workaround for a second missing primitive.

The gap is this: an agent cannot reliably report its own identity, model version, or configuration. These facts exist at the invocation boundary — the moment the infrastructure hands the agent its domain — but no current framework captures them there automatically. The result is that provenance must be recorded by whoever is present at that boundary: the human author who initiated the session, who knows which agent was used and at what settings because they controlled for those things themselves.

This works in a supervised authoring workflow. It does not scale, does not survive delegation, and is not the right place for the responsibility. The infrastructure that brokered the invocation knows what the human knows — and knows it more reliably, at the moment it matters, without depending on the human's presence or attention.

### The Workaround: Provenance Stamping

In the Concept Clusters authoring system, this gap is addressed through a `draft_assistance_stamps` mechanism \[CITATION: jmajerus/concept-clusters, docs/STORAGE-DOMAINS.md\]. Each tool call through the MCP authoring server writes a structured record to a separate table: which tool was called, by which authenticated session, at what time. This is not self-reported by the agent — it is recorded by the server at the invocation boundary, automatically, as a side effect of handling the request.

The stamp record does not capture model version or reasoning configuration, because those are not available to the server from the MCP protocol as currently specified. What it captures is the authenticated session identity and the tool interaction pattern — enough to reconstruct, in combination with the human-recorded provenance entry, a reasonably complete picture of who did what and when.

The human-controlled provenance entry records what the stamp cannot: which agent, which model, which settings. The stamp records what the human cannot do reliably at scale: the precise sequence and timing of infrastructure interactions. Together they approximate what a proper session provenance capture primitive would record automatically and completely.

### The Missing Primitive

What the workaround approximates is **session provenance capture**: automatic, infrastructure-level recording of the identity, configuration, and conditions of every agent invocation that modifies a document domain. Not self-reported, not manually recorded, but captured at the boundary by the infrastructure that enforced the domain scope.

The relationship between the two primitives is precise:

- **Write-domain scoping** answers: *what is this agent authorized to change?*
- **Session provenance capture** answers: *who changed it, and under what conditions?*

The first without the second leaves provenance as a human responsibility. The second without the first means provenance is captured for agents still bearing the full integrity burden. Together they form a complete account of agentic document editing: scoped authority on the write side, automatic attribution on the record side.

Session provenance capture is a companion primitive, not a corollary of write-domain scoping — it requires its own design, its own protocol-level specification for what information is available at an invocation boundary and how it is conveyed, and its own treatment of the privacy and auditability questions that automatic attribution raises. That treatment is deferred here. What this paper contributes is the observation that the two primitives are genuinely paired: a system that implements write-domain scoping but not session provenance capture has correctly located the write responsibility while leaving the attribution responsibility in the wrong place. The Concept Clusters implementation is honest about occupying exactly that intermediate state.

---

## 11. Toward Implementation

This paper argues for a primitive, not a specification. The implementation details — how write-domain annotations are expressed in a schema language, how infrastructure enforces domain boundaries across different document formats and transport protocols, how merge semantics handle conflicts when multiple agents contribute to the same document — are real engineering problems deserving separate treatment.

A few observations are worth making, however, about what an implementation must and must not do.

It must be **infrastructure-level**, not convention-level. A write-domain declaration that agents are expected to respect voluntarily is not write-domain scoping. It is a read-only field convention with better documentation. The enforcement must be mechanical and unconditional.

It must be **invisible to agents**. An agent that knows it is operating in a scoped context — that there are fields it cannot see, that its output will be merged with content from other sources — is an agent that might try to reason about or compensate for its constraints. The domain boundary should be transparent: the agent receives its domain as if it were the whole document, and returns its changes as if it were the whole document. The scoping happens entirely in the infrastructure layer.

It must be **composable**. Real documents have multiple domains with different owners, different lifecycles, and different validation rules. The infrastructure must support composition of domains into complete documents without requiring any single agent or system component to be responsible for the whole.

It must **not require document format changes** in the simple case. A write-domain scoping infrastructure should be able to operate on existing document formats by maintaining domain membership externally — in a schema registry, a capability store, or the infrastructure's own configuration. Retrofitting annotations into every existing document format is not a prerequisite for the primitive to be useful.


### Open Questions

Three questions a technically sophisticated reader will raise deserve direct acknowledgment, even where full answers remain future work.

**What is the relationship of the primitive to storage partitions, and how is that relation abstracted?**

Write-domain scoping is defined at the interface between the infrastructure and the agent — not at the storage layer. The primitive declares domain ownership; storage partitioning is one way to enforce it physically beneath that interface. The abstraction relationship is: the enforcement contract (what the agent sees, what it can modify) should be expressible independently of whether the backing store is a column-partitioned relational database, a document store, a file system, or something else. Section 9 shows database column partitioning as a workable approximation of the primitive; the point is that the agent-facing contract is the same regardless of how the backing store happens to implement it. A full specification of this abstraction — how domain declarations map to enforcement mechanisms across storage types — is engineering work beyond this paper's scope, but the abstraction boundary itself is clear: the primitive lives above the storage layer and below the agent.

**What about handling different backend storage modalities?**

This follows from the first question and asks whether the primitive is genuinely general or only tractable for relational and JSON storage. The answer is that the primitive is general precisely because it is defined at the interface layer. Different backends implement the enforcement differently: a relational store uses column or row partitioning, a document store uses field projection, a file system uses directory isolation or access control lists. The agent-facing contract — here is your domain, everything else does not exist for you — is identical in each case. PatchOptic's future work acknowledges this directly, naming "database updates, file edits, tool calls, and domain-specific operations" as requiring the same source-read discipline as its JSON Patch prototype. The generalization across storage modalities is a motivation for elevating the mechanism to a primitive rather than leaving it as a workflow-step contract tied to one storage representation. Each backend requires its own enforcement adapter; the primitive provides the contract those adapters implement.

**Is having reference implementations enough?**

No — and the paper should be honest about this. Reference implementations are necessary but not sufficient. They demonstrate that the primitive is real and implementable, which is the first condition for broader adoption. But a primitive becomes infrastructure only when it is adopted by the frameworks, protocols, and platforms that practitioners use by default — not when a handful of implementations exist that practitioners must choose to adopt. The targets are framework and platform designers: the MCP specification, agent orchestration frameworks such as LangChain and LlamaIndex, document database APIs, and cloud document editing platforms. Reference implementations are the proof of concept that precedes that adoption and makes the case that the primitive is tractable. They are the beginning of the path, not its end. The paper's call to action is therefore directed primarily at infrastructure and framework designers, not at application developers who will otherwise continue building one-off workarounds in the space where the primitive should be.


---

## 12. Conclusion

The integrity burden is real, it is compounding, and it is not solvable within the round-trip model of agentic document editing. Every workaround that reduces it does so by approximating a capability the infrastructure does not provide — and adds complexity that must be maintained indefinitely.

The missing primitive is write-domain scoping: a formal, infrastructure-enforced boundary around what an agent owns, rendering everything else invisible and immutable by construction. This primitive correctly locates responsibility, enables composable multi-agent document workflows, and makes human review meaningful by eliminating a class of mechanical errors that should never have been human concerns.

The stakes are not merely efficiency. In observed authoring practice, the integrity burden has proved the difference between a successful session and a failed one — not a performance penalty but a reliability threshold. Freeing an agent from obligations outside its domain is not a marginal improvement; it is the condition under which the agent can reliably do what it is actually for.

The workarounds will continue to accumulate until the primitive exists. The question is not whether write-domain scoping is the right direction — it is whether the infrastructure community will build it deliberately or arrive at it incrementally, one workaround at a time.

---

*The author developed these ideas during work on the Concept Clusters authoring project \[CITATION: jmajerus/concept-clusters\]. The working implementation of the database partitioning approach described in Section 8 is documented at \[CITATION: jmajerus/concept-clusters, docs/STORAGE-DOMAINS.md — to be created\]. This paper was drafted with the assistance of Claude. The ideas are the author's own.*
