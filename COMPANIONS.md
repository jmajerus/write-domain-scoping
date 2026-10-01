# Associated documents

Companion notes and implementation references for
[*The Integrity Burden: Why Agentic Document Editing Belongs in the
Infrastructure, Not the Agent*](README.md).

The paper states the thesis. These documents hold arguments and working
evidence that should not inflate the main text, but remain part of the
public record for the position.

## Companion arguments (this repository)

| Document | Role |
|---|---|
| [Guidance specialization is not a storage partition](companions/guidance-specialization-vs-storage-domains.md) | Guidance vs ownership axes; why profiles-everywhere cannot replace write domains; when a further domain is justified (the shelf) and when it would only fragment the contract; write-once as a lifecycle constraint inside a domain, not a new partition; Concept Clusters as illustration, with higher-stakes settings (clinical, claims, legal, CMS, …) where the case is clearer |
| [Clinical documentation, against the record systems](companions/clinical-documentation-prospective.md) | The clinical claim checked against OpenMRS, OpenEMR, and VistA-family TIU: the problem list is not the note, an encounter save can drop a link without deleting the issue, and pulling a list into note text does not make that list part of the note |
| [A page and a post](companions/page-and-post.md) | The same claim where the writes are obvious: MediaWiki will not let a text edit change protection, and forgetting a restriction on protect removes it; WordPress still accepts the post body and its publish status in one update |
| [One document, several writes](companions/one-document-several-writes.md) | Several write domains inside one document: a Jupyter notebook's sources, outputs, and kernel, which Jupytext already reassembles; and a Concept Clusters puzzle's board, shelf, and lesson, which stay one published document |

## Working implementations (Concept Clusters)

These live in the product repository that motivated the paper. They are the
concrete column-level and provenance instantiations discussed in the paper’s
implementation sections.

| Document | Role |
|---|---|
| [STORAGE-DOMAINS.md](https://github.com/jmajerus/concept-clusters/blob/main/docs/STORAGE-DOMAINS.md) | Write-domain scoping as implemented at the Concept Clusters authoring boundary |
| [PROVENANCE-STAMPS.md](https://github.com/jmajerus/concept-clusters/blob/main/docs/PROVENANCE-STAMPS.md) | Invocation-boundary assistance stamps (companion primitive, partial capture) |

## Bibliographic references

Cited sources for the paper are listed in [REFERENCES.md](REFERENCES.md).

*These companion notes were drafted with the assistance of Grok. The ideas are the author's own.*
