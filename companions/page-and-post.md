# A page and a post

*Companion to [The Integrity Burden](../README.md) and to
[Guidance specialization is not a storage partition](guidance-specialization-vs-storage-domains.md).
The [clinical note](clinical-documentation-prospective.md) is a real split
that takes a record model to see. This one is the same claim on documents
a reader already knows how to tell apart: the words, and whether the page
is locked or the post is published.
[Concept Clusters](https://github.com/jmajerus/concept-clusters/) remains
the authoring system that was built.*

## The wiki already split the writes

A MediaWiki page has text. It also has restrictions: who may edit it, and
who may move it. Those are not two fields of one save.

Changing the text is `action=edit`. The request carries the new text, and
optionally which section. It does not carry a protection level. If the page
is protected against that user, the edit is rejected (`protectedpage`). The
edit does not unlock the page in order to succeed.
([API:Edit](https://www.mediawiki.org/wiki/API:Edit))

Changing who may edit or move the page is `action=protect`. It requires the
protect right and a protect token. The text is not a parameter.
([API:Protect](https://www.mediawiki.org/wiki/API:Protect))

An agent asked to fix a paragraph is holding an edit. Locking or unlocking
the page is not in that request. The software will not accept it there.
That is a write domain in the plain sense: this action writes the words,
and it cannot write the policy.

Protect has the other half of the paper's omission rule, and it is
documented in one sentence. The protections parameter is the whole set.
"Any actions not listed will have restrictions removed." Sending
`edit=sysop` and forgetting `move=sysop` does not leave the move
restriction alone. It lifts it. The infrastructure treats that request as
a replacement of the policy, not as a hint about one action. An agent that
round-trips protection has to reproduce every restriction it did not mean
to change. An agent that only edits never enters that request.

## The post still joins them

A WordPress post update is one write. `POST /wp/v2/posts/<id>` accepts
`content` and `status` together. Status is one of `publish`, `future`,
`draft`, `pending`, or `private`.
([Posts](https://developer.wordpress.org/rest-api/reference/posts/))

A request that corrects a sentence can also set `status` to `publish`.
Nothing in the shape of the update says the wording and the publication
are different decisions. Some roles lack the capability to publish, and
the API will reject those callers. That is a permission check on the whole
request. It is not a projection that withholds `status` from an agent
whose job is the sentence. A caller who is allowed to do both is allowed
to do both in the same body, including by accident, because both fields
were in the document they were handed and both fields were in the document
they returned.

The contrast is the point. MediaWiki will not let a text edit change who
may edit. WordPress will let a content update change whether the post is
public, when the caller is allowed to publish at all. The first has drawn
the domain. The second still asks the agent to carry publication state
through a wording change.

## What the agent should be given

For the wiki, the wording pass receives the text. Protection is not in the
payload, so it cannot be dropped or relaxed by an incomplete reply. A
protection pass, if one is authorized, receives the restrictions and must
send the whole set, because omission removes. It does not receive the
page text as something it has to rewrite in order to lock the page.

For the post, the wording pass should receive the content. `status` stays
where it is. Publishing is a different write, made by whoever is allowed
to publish, not a field that rides back with the corrected sentence. The
WordPress API does not offer that split today. The wiki shows that the
split is an ordinary shape for software, not a special case of clinical
records.

Headings inside the page are not further domains. `action=edit` can target
a section, and the page is still one text write. A domain per section
would make the agent reassemble the article. The seam that is already
visible is text versus protection, and body versus publish state. Several
writes inside one file — a notebook's sources and outputs, a puzzle's
board, shelf, lenses, and lesson — are
[One document, several writes](one-document-several-writes.md).

## What is not claimed

MediaWiki's edit and protect actions are not a general publishing
platform. WordPress's role checks are real; this note does not say a
contributor can publish by putting `status` in the body. It says that when
the caller may both edit and publish, those two decisions share one
update. That is enough. One system already refuses to let a wording change
unlock the page. The other still lets a wording change publish the post.
