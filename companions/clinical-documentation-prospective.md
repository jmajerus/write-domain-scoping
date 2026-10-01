# Clinical documentation, against the record systems

*Companion to [The Integrity Burden](../README.md) and to
[Guidance specialization is not a storage partition](guidance-specialization-vs-storage-domains.md).
This note is not a theory of every chart. It is what three public systems
already do, in their own documentation and, where it matters, in their
source. [Concept Clusters](https://github.com/jmajerus/concept-clusters/)
remains the authoring system that was built. These are the clinical records
the ownership claim can be checked against.*

## The records are already separate

**OpenMRS** states the split in its information model. A problem list is
maintained by hand, apart from observations. Recording that a patient has
"Diagnosis Present = Diabetes" does not add diabetes to the problem list. An
observation is one moment. A problem is longitudinal: it has a start and an
end. The allergy list is the same kind of record. Observing a reaction does
not add the allergy. ([OpenMRS Information Model](https://guide.openmrs.org/getting-started/openmrs-information-model/))

**OpenEMR** stores medical problems, allergies, medications, surgeries, and
dental issues as patient issues, on the patient summary, not as the text of
an encounter. The user guide's reason for that shape is the link: one issue
can be associated with many encounters, and one encounter with many issues.
The encounter form does not contain the issue. It records which issues
belong to this visit. ([OpenEMR 4.1 Users Guide](https://www.open-emr.org/files/OpenEMR_4.1_Users_Guide_V1.pdf),
"Adding Issues" and "Associating Issues with Encounters")

The current source still draws that line in the save. Saving an encounter
calls `replaceIssuesForEncounter`. An issue id that is no longer posted is
unlinked: the row in `issue_encounter` is deleted. The issue itself lives in
`lists` and is updated by a different write, `updateIssue`. Dropping an
issue from the encounter payload drops the association for that encounter.
It does not delete the problem, the allergy, or the medication.
([`PatientIssuesService.php`](https://github.com/openemr/openemr/blob/master/src/Services/PatientIssuesService.php),
called from the encounter save)

A prescription makes the same distinction on the way in. "Add to Medication
List" is a separate yes on the prescription form. Saying yes also creates a
medication issue. The prescription is not the list.

**VistA CPRS** puts Problems and Discharge Summaries on different chart
tabs. A discharge summary is a document. The problem list is not a section
of it. ([CPRS Clinician's Getting Started Guide](https://www.va.gov/vdl/documents/Clinical/Comp_Patient_Recrd_Sys_(CPRS)_Archive/cprslmum.pdf))

The pull-in the chart actually offers is documented on the VistA text
system those notes use. In the RPMS EHR, which trains clinicians on that
Text Integration Utility, a data object extracts data from another package
and inserts it into the boilerplate of a progress note or a discharge
summary. The placeholder sits in the template. Medications, allergies, and
the problem list are among the things those objects are there to pull. What
lands in the note is text. The source of that text remains the other
package. ([TIU Data Objects](https://www.ihs.gov/rpms/training/course-materials/?download=1&flname=Tab10_TIU_Data_Objects_LR+2017.pdf&p=rpmstraining%5CHIM+CAC%5CTab10_TIU_Data_Objects_LR+2017.pdf),
IHS RPMS training)

## What the agent is holding

An agent asked to revise a discharge summary that already expanded a
problem-list object is holding note text. The problem list is a different
record, which the template copied from. Editing a line of that inserted
text edits the summary. It is not, on the mechanism above, an edit of the
problem-list package. The copy and the source have already been separated
by the software that did the pull.

An agent asked to save an OpenEMR encounter while "tidying" the linked
problems is holding a list of issue ids. Omit one, and the encounter save
unlinks it. The issue row is untouched, because that save never called the
issue update. An agent that then rewrites the issue title, because the title
was visible beside the encounter, has taken a second write the encounter
form does not perform. OpenMRS is stricter still: a diagnosis observation
and a problem-list entry are not the same write, and the system will not
infer one from the other.

That is the integrity burden in a form a record system already recognizes.
The note, the encounter link, and the longitudinal list are different
saves. A profile that says "improve this discharge summary" and then
round-trips every list the summary can display asks the agent to keep
straight a set of boundaries the applications drew on purpose.

## What write-domain scoping claims here

A save of the note writes the note. A save of the problem list writes the
problem list. A save of the encounter writes the encounter and, in
OpenEMR, the links that encounter is allowed to replace. Pull-in, where TIU
does it, is an insertion into the note. It is not a merge of the two
records into one document the agent must return intact.

The agent authorized to edit the summary receives the summary. The expanded
object text may be in it. The problem-list package is not the writable
document. The agent authorized to maintain the problem list receives the
problem list. The summary that quoted it earlier may be context. Saving the
list does not rewrite that earlier snapshot. If both should change, that is
two authorized writes, which is what these systems already require of a
person.

Inside the note, the template's headings are still one document. A SOAP
boilerplate is one TIU document even when it pulls medications into the
objective section. The seam that pays for itself is the one the record
already has: the other package, and the link table. A domain per heading
would invent structure these systems do not have.

## What is not claimed

These three systems do not become a specification for any other product.
OpenEMR's encounter save is allowed to change links, and that is a real
write, scoped to `issue_encounter`. TIU's documented operation is insertion
into the note; this note does not go on to claim how every later edit of
the expanded text behaves in every site's template. The argument does not
need that. The split between the note, the link, and the list is already
in the software, and an agent that is handed all three as one document has
been given a job the record system refused to make one job. The same refusal,
on a document that does not require a chart to see, is
[a page and a post](page-and-post.md).
