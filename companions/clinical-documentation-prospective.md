# Clinical documentation, prospectively

*Companion to [The Integrity Burden](../README.md) and to
[Guidance specialization is not a storage partition](guidance-specialization-vs-storage-domains.md).
This note is a prospective scene. It is not an implementation, not a
proposed electronic-health-record schema, and not clinical, regulatory, or
privacy advice. [Concept Clusters](https://github.com/jmajerus/concept-clusters/)
remains the system that was built. Clinical documentation is where the same
ownership test is easier to see, because a wrong byte is not a bad puzzle.*

The table in the guidance companion gives the clinical row in one line: a
coding profile must not rewrite medications or a signed narrative, and
omitting a problem must not delete it. This note is that line opened up, so
the cut can be inspected. It uses a discharge summary because that document
already has several owners in ordinary hospital practice. The names below are
the decisions, not a field list to implement.

## One document, several decisions

A discharge summary is one chart artifact. It is not one write.

**Narrative.** What happened, what was concluded, and what the patient was
told: the hospital course, the assessment, the instructions in the
clinician's words. This is the judgment a prose pass is for.

**Medication list.** The reconciled list: drug, dose, route, status. It is
often decided in a different pass from the course, sometimes by a different
role, and a sentence in the course that mentions a drug is not the list.
Omitting the list from a prose save must not discontinue a medication.

**Codes.** Diagnoses and procedures as coded facts for billing and retrieval.
Assigning them is a classification judgment of a different kind from telling
the hospital course. A coder's pass is not an invitation to improve the
prose, and a prose pass is not competent to invent a billable code because
the field was sitting in the document.

**Filing.** Which problems this note is associated with, and which of those
are active for this encounter. This is the shelf: where the note lives in
the chart. It is not the course, and it is not the code set. Changing the
filing should not require rewriting the course. Editing the course should
not drop a problem because the problem list was absent from the payload.

**Attestation.** Who signed, when, and that this version is the signed one.
The agent does not author this. After signature, the signed narrative is
write-once: an addendum is a new write, not a silent rewrite of the signed
bytes.

**System.** Patient, encounter, and document identifiers, revision tokens,
hashes, and the record's lifecycle. The agent does not see them. Reproducing
an encounter id "helpfully" is the same failure as reproducing a puzzle id.

Note type and specialty are not on this list. "Discharge summary" and
"cardiology" are briefs. They change what the agent is asked to judge. They
do not change who must preserve which bytes.

## The round trip that fails

A coding assist is asked to tighten the coded diagnoses on an already signed
discharge summary. Under the round-trip model it is handed the note. It
returns the note. The codes are better. Also, a sentence in the hospital
course has been smoothed, a medication the model judged redundant is gone,
and a problem that was not restated in the payload is gone. Each of those
edits can look like care. Each is an integrity failure in a domain the task
did not own. Validation after the fact can catch some of them. It cannot
make the agent a reliable custodian of the fields it had to carry in order
to be allowed to save.

A profile named coding-assist, with a phase schema of the code fields and a
paragraph that says "do not rewrite the course or the medication list," is
the workaround the paper already criticizes. Compliance is probabilistic. It
gets worse as the note gets longer and as the number of "do not touch" rules
grows. Omission stays ambiguous: the code view did not include the
medication list, and nothing in the profile makes preservation of that list
a server obligation.

## What the infrastructure would enforce

The agent receives the domain it is authorized to write, plus read-only
context for the decisions it must see and must not save.

A coding pass receives the codes as the writable projection. The course, the
medication list, and the filing are context. Omitting a medication cannot
discontinue it, because the medication list is not in the write. Smoothing a
sentence in the course has no effect, because the course is not in the
agent's possession. The same pass can still read the course. Coding without
the course would be a worse judgment, not a purer domain.

A medication-reconciliation pass receives the medication list. It does not
receive the signed course as a writable document. Guidance may say that a
changed medication should be reflected in the patient instructions. That is
a brief spanning two domains, which is legitimate. The medication save still
does not replace the narrative. If the session is also authorized to change
the instructions, that is a separate narrative write. The span is in the
guidance. The seam is in the save.

A filing pass receives the problem associations and their active status,
with the note's identity as context, not the course and not the codes.
Re-filing many notes is one infrastructure call over that projection. It is
not a per-note rewrite of the hospital course. This is the same payoff as a
shelf move in the built example: the later edit that is *about* placement
does not carry the rest of the document, and the later edit that is *not*
about placement cannot drop the placement.

A first composition can still be one complete document, when the session is
authorized for every clinical domain it is authoring. The server partitions
on store. Later passes do not reassemble the chart in the agent's context.
Signature is not part of that composition. It is infrastructure, and once
applied it freezes the signed narrative against later authored saves.

## Where the cut stops

The failure mode to avoid is a domain per section of the form. History,
exam, assessment, and plan are one narrative decision even when the screen
has four headings. A phase can focus the agent on the assessment. The save
still owns the narrative. Splitting those headings into columns would make
the agent keep a mental index of fragments and reconstruct the note before
it could write a paragraph. That is the integrity burden in a new costume.

The same stop applies inside the other decisions. The medication list is one
domain, not a domain per drug. The code set is one domain, not a domain per
vocabulary. Filing is one domain, not a domain per problem. The test that
justified the four clinical seams above is the same test that refuses a
fifth: a further domain pays for itself only when a later pass must not
carry or clobber that decision, and when changing that decision must not
require the other passes. A heading inside the course fails the test. The
medication list passes it.

## What this does not claim

- It does not claim a hospital should store a discharge summary as six JSON
  columns, or that these names match any particular record system.
- It does not claim an agent should sign, code, or discontinue medications.
  Authority to write a domain is a clinical and legal question. The
  primitive only says that whoever is authorized writes that domain and no
  other.
- It does not claim profiles are useless here. Discharge-summary,
  specialty, and coding-assist briefs are how the judgment gets specific.
  They remain briefs. They do not become the integrity mechanism.
- It does not claim the other rows in the guidance companion are thinner
  problems. Claims, contracts, scientific reporting, and editorial systems
  have the same shape. This note opens one row so the shape can be checked
  against a document people already know has more than one owner.
