# One document, several writes

*Companion to [The Integrity Burden](../README.md) and to
[Guidance specialization is not a storage partition](guidance-specialization-vs-storage-domains.md).
The [clinical records](clinical-documentation-prospective.md) are several
documents. The [page and the post](page-and-post.md) are two actions, one
of which a wiki already separated. This note is several writes inside one
document, which is the shape the paper argues for.
[Concept Clusters](https://github.com/jmajerus/concept-clusters/) is the
authoring system that was built that way.*

## The notebook is one file

A Jupyter notebook is one JSON document. The format gives it metadata, a
format version, and a list of cells. A code cell holds the source the
author wrote, the outputs produced by running that source, and an
execution count. The notebook metadata holds the kernel the document is
for. ([The Notebook file format](https://nbformat.readthedocs.io/en/latest/format_description.html))

Those are different owners, and they sit in the same file. The source is
the author's. The outputs are the kernel's. The kernel specification is
the environment's. An agent asked to rename a variable is asked for the
first. Under a round trip it is handed all three, and it has to return all
three. A cleared output, a renumbered execution count, or a missing kernel
is not a worse version of the rename. It is a write in a part of the
document the rename did not own.

## The tool that already splits them

Jupytext saves that same notebook as text, a `.py` or a `.md` file. The
text contains the inputs, and selected metadata. It does not contain the
outputs. Pair the text with the `.ipynb` and the outputs stay in the
notebook file. On open, the inputs are read from the newer text file and
the outputs are read from the `.ipynb`. The notebook the author sees is
assembled. The person editing the text never had the outputs to preserve.
([Text notebooks](https://jupytext.readthedocs.io/en/latest/text-notebooks.html),
[Paired notebooks](https://jupytext.readthedocs.io/en/latest/paired-notebooks.html))

That is more than one write domain in one document. The text write replaces
the sources. The output write is whatever last ran the kernel, and it is
restored rather than reproduced. Omission of an output from the text file
does not delete the output. The infrastructure still has it.

Cells are not further domains. A notebook has many cells and one source
write. Splitting each cell into its own save would make the author
reassemble the notebook in order to change one of them. The seam that pays
for itself is source versus output versus the kernel the document requires.

## Several authored decisions, still one document

Source versus output is an easy owner to see, because one of them is not
the author. A document can also contain several decisions that *are*
authored, and that still must not ride on each other's saves.

A Concept Clusters puzzle is one document. Four of its writes are agent
writes, and they are different decisions. The board is the clusters, the
terms, and the bridges. The shelf is where the puzzle is filed and found:
primary category, membership, subcategory placement, search tags, and
optional level. Pedagogy is the lenses, bridge relationship annotations,
related puzzles, and language. Lesson is the introductory essay before
play, read against the subject with the lenses withheld, and allowed to
merge cited works into the puzzle's single `info.citations` bibliography.
Provenance and system state are not agent writes. The stored form keeps
the agent writes in separate projections. A pedagogy save does not receive
the lesson. A lesson save does not receive the lenses. A shelf save does
not receive the board. The published puzzle is still one document,
reassembled by the server.
([STORAGE-DOMAINS.md](https://github.com/jmajerus/concept-clusters/blob/main/docs/STORAGE-DOMAINS.md))

The first write can still be the whole document. The server files each
part in its projection. Later, the pass that is not about the shelf does
not carry the shelf, the pass that writes prose does not carry the lenses,
and the pass that edits lenses does not rewrite the essay. That is the
same assembly Jupytext does for outputs, applied to decisions the agent
does make, and must not make all at once.

Related puzzles and language stay in pedagogy, and citations stay one
puzzle-level list rather than a second bibliography on the lesson. A
further cut would pay for itself only if a later pass had to stop
carrying that decision, and a change to that decision had to stop
rewriting the others. Short of that, another projection is another
fragment the agent has to remember.

## What is not claimed

Jupytext does not withhold outputs from someone who edits the `.ipynb`
directly. The default notebook file is still a round trip. The text pairing
is the split. Concept Clusters is one author's system, not a general
document runtime. What both show is the shape: one document, more than one
write, and a save that replaces only the write it was given.
