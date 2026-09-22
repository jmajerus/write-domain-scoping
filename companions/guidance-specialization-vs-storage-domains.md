# Guidance specialization is not a storage partition

*Companion to [The Integrity Burden](../README.md). [Concept Clusters](https://github.com/jmajerus/concept-clusters/)
is one **illustrative** working system — useful because the axes were forced
into contact there — not the canonical application of the thesis.
Implementation notes for that illustration:
[STORAGE-DOMAINS.md](https://github.com/jmajerus/concept-clusters/blob/main/docs/STORAGE-DOMAINS.md).*

## Claim

Write-domain scoping and authoring-profile guidance are two different levers
for reducing agent burden. They must not be collapsed into one mechanism.

- **Domains** answer: *who owns this field, and who is responsible for keeping
  it intact across a write?*
- **Profiles** answer: *given that ownership, what design brief should the
  agent follow for this class of document or task?*

Specializing guidance is a legitimate way to shrink the *judgment* surface.
It is not a reason to invent a new storage domain, duplicate field owners, or
fork the edit toolset.

Concept Clusters shows the distinction in a live authoring stack. Higher-
stakes domains make the same distinction *justifiable* more quickly: when
round-trip corruption is a compliance, safety, or liability event, “more
profiles” is an obviously inadequate substitute for owned seams.

## Why the confusion arises

Both moves look like “give the agent less.” Domain projections hide or
protect fields the agent should not carry. Profile briefs replace generic
authoring advice with a compact genre-specific route (for example a
vocabulary-teaching brief versus a quiz-led brief in an educational
authoring system).

The deeper confusion is structural, not just rhetorical. Write domains are
often implemented as **substructures of a later-materialized full schema**:
the agent writes one owned slice, and infrastructure reassembles the
canonical document. Profiles and phases also surface **sub-schemas** to
agents: a focused JSON Schema projection, a phase brief, a genre overview.
From the outside, both look like “part now → whole later.”

That analogy is real and useful for pedagogy. It is also where axes get
collapsed:

| Looks similar | Domain projection | Profile / phase sub-schema |
|---|---|---|
| Agent sees a partial document shape | Yes | Often yes |
| Full schema is validated later | Yes (after reassembly) | Yes (after the pass) |
| Purpose of the cut | Ownership / integrity | Judgment / task focus |
| Durable across saves? | Yes — sibling domains are stored and merged | No — request-time view; does not own fields |
| Omission means | Within a domain replace, may mean removal; outside the domain, preserve | Preserve (phase is not a whole-document replace) |
| Spans domains? | No — a domain *is* the cut | Yes — a profile may brief several domains together |

So: **same compositional shape, different persistence and responsibility.**
A domain substructure is a storage seam. A profile or phase sub-schema is a
working view over seams that already exist. Treating the second as if it
were the first is how a genre of work (“quiz-led,” “discharge summary,”
“amendment package”) gets mistaken for a new write domain.

If those are treated as the same kind of partitioning, the natural (and
wrong) follow-ups appear:

1. Add a genre write domain beside the integrity domains.
2. Give each document genre its own save tools or storage column set.
3. Treat a profile selector as a storage selector rather than a guidance
   selector.
4. Treat a phase or profile schema as a standalone replaceable document
   because it *looks* like a domain projection.

Each of those couples a *genre or workflow* to an ownership cut. Genres and
passes change with product taste and institutional process; ownership cuts
should change only when integrity responsibility moves.

The practical test: *if the agent omits a sibling field, does infrastructure
still owe that field’s preservation?* If yes, you are in domain territory.
If the field was merely out of the current brief or phase view, you are in
guidance territory — and omission must not become deletion.

## Orthogonal axes

A mature agent-authoring boundary separates selectors of which only one is a
storage partition:

| Axis | Question | Stored on the document? | Changes ownership? |
|---|---|---|---|
| Write domain | Who owns this field across saves? | Yes (projections / columns / capabilities) | Yes |
| Pass / phase | Which slice of a domain is in scope now? | No (request argument) | No — phase binds to a domain |
| Authoring profile | Which design brief applies? | No (request argument) | No — may *span* domains |
| Document kind / type | Which authored type is this? | Often yes (content-owned metadata) | No |

A profile may span existing domains without becoming one. Guidance can ask
an agent to co-design material that will later be saved through more than
one owned surface. Ownership stays on the integrity domains; only briefs
and, where useful, type-specific schema constraints change.

Document kind or type metadata records specialization on the document. It
is not a third projection. Browse taxonomy, product line, or folder
hierarchy remains a separate axis from write ownership.

### Illustration: Concept Clusters

In Concept Clusters, the map is concrete: `content` / `pedagogy` /
`provenance` / `system` domains; MCP `profile=` for vocabulary-context or
trivia-quiz briefs; `puzzleKind` as content metadata; category as browse
taxonomy. Profiles there span content and pedagogy without new columns.
That system motivated the paper; it is evidence that the axes collide in
practice, not a claim that educational puzzles are the primary market for
the primitive.

## Other applications (where the stakes clarify the case)

The Integrity Burden is easiest to dismiss when the only harm is a broken
puzzle draft. It is harder to dismiss when out-of-domain corruption is a
regulatory, clinical, or contractual failure. Profiles still help in these
settings — “discharge summary,” “amendment redline,” “adverse-event
narrative” — but they cannot be the integrity mechanism.

| Setting | Judgment profiles (examples) | Integrity domains (examples) | Why domains are non-optional |
|---|---|---|---|
| Clinical documentation | Note type, specialty brief, coding assist | Clinician narrative vs coded/billing fields vs medication list vs audit/provenance vs EHR system ids | A coding profile must not be allowed to rewrite meds or silently alter signed narrative; omission ≠ deletion of problem lists |
| Insurance / claims | FNOL intake, adjuster summary, SIU review | Claimant narrative vs coverage determination vs payment instructions vs fraud flags vs system case ids | Guidance for “write a clearer loss description” must not round-trip payment routing or alter locked coverage decisions |
| Legal / contracts | Clause style, jurisdiction pack, playbook profile | Negotiated terms vs boilerplate library vs signature/notary block vs matter metadata vs DMS versioning | A “friendly tone” profile rewriting commercial terms must not touch signature blocks or matter identifiers |
| Scientific reporting | Paper section, grant narrative, lab-notebook entry | Results prose vs methods/data references vs authorship/ORCID vs funder compliance vs repository ids | Section profiles improve writing; they must not invent coauthors or mutate dataset DOIs |
| Software change / RFC | Feature brief, security review, ops runbook | Spec normative text vs ADRs vs approval state vs ticket linkage vs deploy metadata | An implementation profile must not clear approvals or rewrite linked incident ids |
| Media / CMS | News desk, SEO pack, localization | Body copy vs rights/licensing vs embargo/publish state vs asset ids | SEO guidance must not alter license grants or un-embargo by omission |

In each row, **profiles proliferate naturally** (many desks, specialties,
playbooks) while **domains stay few and stable** (narrative judgment vs
structured operational truth vs attribution vs system envelope). That is
the scaling argument the paper needs: genre guidance is unbounded;
ownership seams are not.

Proposable near-term systems need not wait for a universal document
runtime. Column-level or capability-gated APIs in any one row above already
justify the primitive more sharply than educational authoring alone —
Concept Clusters remains the worked example of *having built* the
approximation, not the existence proof that the problem matters.

## What each lever is for

### Write domains (integrity)

The Integrity Burden thesis: round-trip editing makes the agent responsible
for information it neither needs nor is competent to maintain. Domains move
that responsibility into infrastructure. Focused reads and saves, protected
provenance, and deferred materialization are consequences of ownership.

Domains are the wrong place to encode “this document teaches vocabulary,”
“this note is a discharge summary,” or “this pass is SEO-only.” Those are
judgments about *what to author*, not about *who must preserve which bytes*.

### Profiles (judgment)

Profiles reduce simultaneous design confusion for specialized genres and
roles. They select compact overviews and phase briefs, and they may steer
workflow (integrated co-design versus staged inventory; specialty note
versus coding assist). They do not change which fields the server will
accept, preserve, or reject on a focused save.

Soft schema asymmetry is allowed: one genre may hard-enforce a document
shape while another leaves a mode convention as guidance. That unevenness is
a product choice about fail-closed genre rules, not evidence that the softer
genre needs its own storage domain.

### Kind-specific schema rules (document constraints)

When a kind has a mechanically distinct shape, express it as a constraint on
the shared canonical schema, not as a new domain. Constraints answer “is
this document valid for its declared kind?” Domains answer “may this agent
replace this field?”

## Failure modes this distinction prevents

1. **Domain proliferation** — every new genre gets columns, merge rules, and
   migration cost without a change in integrity ownership.
2. **Tooling forks** — genre-specific surfaces that reimplement optimistic
   concurrency, provenance protection, and publication for small guidance
   differences.
3. **False completeness** — treating a profile brief as a whole-document
   replacement contract, or treating a phase projection as a domain.
4. **Cross-axis conflation** — agents set taxonomy category instead of
   document kind, or request a profile without recording the kind, because
   the system appeared to treat those as interchangeable selectors.

Keeping the axes separate makes each failure diagnosable: wrong guidance is
a profile/skill problem; lost sibling fields are a domain/merge problem;
invalid genre shape is a schema/kind problem.

## Why profiles cannot replace write domains

The converse mistake is as important as inventing genre domains: treating
**profiles everywhere** as a substitute for write-domain scoping. If every
authoring situation gets a specialized brief, a phase schema, and
instructions about what not to touch, it can look as though the integrity
problem has been designed away. It has not. Profiles remain in the class of
workarounds the Integrity Burden paper criticizes — they reduce how often
agents stumble; they do not move integrity into infrastructure.

### What “profiles everywhere” tries to do

Scale the guidance lever until every task has a compact contract:

- a genre or role profile for each document type,
- a phase sub-schema for each pass,
- prose rules for fields outside the current focus,
- validation after the fact to catch round-trip damage.

That stack improves judgment. It does not declare ownership, withhold
out-of-domain bytes, or merge sibling domains by construction. The agent
still receives (or can request) a complete-enough document, still returns a
document-shaped payload, and still bears probabilistic responsibility for
everything it saw.

### Limits that do not go away with more profiles

1. **Enforcement stays probabilistic.** A profile can say “do not rewrite
   provenance” or “preserve lenses outside this pass.” Compliance degrades
   with context length, task complexity, and the number of simultaneous
   conventions — the same failure mode as any other instruction-only
   boundary. Write domains remove the fields from the agent’s possession;
   profiles ask the agent to remember not to touch them.

2. **Omission semantics stay ambiguous.** Profile and phase sub-schemas look
   like partial documents. Without infrastructure merge rules, agents and
   callers treat “not in this view” as “delete,” “leave unchanged,” or
   “invent a default” with no stable answer. Domains make preservation of
   sibling fields a server obligation; profiles cannot.

3. **Profile combinatorics replace domain composition.** Every new genre ×
   pass × role multiplies briefs and projected schemas that must stay
   consistent with the canonical document. Domain composition scales by
   ownership (few seams, many documents). Profile multiplication scales by
   workflow taxonomy (many seams that do not own anything). The complexity
   budget moves from integrity repair into guidance maintenance — still not
   spent on judgment quality.

4. **Protected and system fields never become safe.** Attribution, revision
   tokens, hashes, and lifecycle state are exactly the fields agents are
   worst at round-tripping. No amount of genre briefing makes an agent a
   reliable custodian of those. They require domains the agent cannot write
   (and ideally cannot see).

5. **Complete-path escape hatches reintroduce the full burden.** Real
   systems keep a complete-document save for compatibility or recovery. If
   integrity depends on always selecting the right profile, one unprofiled
   or `complete` write restores the round-trip model. Domain-scoped
   infrastructure still protects out-of-domain fields on that path; a
   profiles-only design cannot.

6. **Cross-domain briefs expose the gap.** Profiles that *span* domains
   (co-designed educational board and lenses; clinical narrative plus coding
   suggestions in one assist) are good for judgment and bad as integrity
   partitions: the brief asks for simultaneous authorship while the return
   object, if unconstrained, still risks corrupting sibling and protected
   fields. Domains let the agent think across domains in guidance while
   writing through owned surfaces; profiles-alone collapse that back into
   one round-trip.

### Profiles as a foil, not a rival primitive

For the paper’s thesis, profiles are a useful foil: they show a second lever
that genuinely helps agents, shares the “part now → whole later” shape with
domain projections, and still fails as an integrity mechanism for the same
reasons phased schemas and read-only conventions fail. Expanding profile
coverage is not progress toward write-domain scoping; it is progress toward
better guidance *beside* write-domain scoping.

The design rule follows: **use profiles liberally for judgment; never use
them as the place integrity lives.**

## What this does *not* claim

- It does not claim profiles are optional decoration. Integrated co-design is
  a real authoring route; it simply must not rewrite the ownership map.
- It does not claim “fewer profiles” is the goal. Specialized briefs remain
  valuable; the limit is using them *instead of* ownership enforcement.
- It does not claim all genre rules must stay advisory. Hard kind constraints
  belong in the shared schema when invalid documents are not useful drafts.
- It does not claim agents will never use a complete-document path.
  Compatibility paths remain; the thesis is about where integrity *belongs*,
  not about deleting escape hatches on day one.
- It does not claim every future specialization fits the existing integrity
  domains. A new domain is justified only when a new *owner* appears — for
  example a class of fields that agents must not see or must never
  round-trip — not when a new teaching genre appears.

## Relation to the main paper

The Integrity Burden paper argues for write-domain scoping as an
infrastructure primitive and notes phased schemas among the workarounds that
approximate it. Concept Clusters appears there as a worked storage
approximation and provenance illustration. This companion asks the paper’s
document set to treat that system as **evidence of construction**, while
using higher-stakes settings to justify *why* the primitive matters and to
show that profile proliferation is the wrong scaling strategy.

The sharpened distinctions for a later paper distillation:

1. Guidance specialization must not be mistaken for a storage partition —
   same compositional shape, different persistence and responsibility.
2. Proliferating profiles cannot discharge the integrity burden.
3. Domains stay few and stable; profiles and playbooks multiply — clinical,
   legal, claims, and CMS settings make that asymmetry obvious.

This document holds the full argument so the paper can stay tight.

## One-sentence form

**Domains partition integrity; profiles specialize judgment; kinds constrain
documents — and mistaking guidance specialization for a storage partition,
or profiles everywhere for write-domain scoping, is how agent-authoring
systems grow the wrong seams.**
