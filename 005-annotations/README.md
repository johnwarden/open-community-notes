# ATProto Annotations Proposal


## Overview

* Any labeler can also act as an **annotator**.
* A label record may include an optional `ref` field pointing to an atproto annotation record.
* `ref` contains a URI (weak ref) of an atproto record of any type.

  * **Coherence/supersession:** for a given labeler `src`, the tuple `(uri, val)` has a single coherent annotation reference at a time (i.e., one current `ref`). When multiple label records exist for the same `(src, uri, val)`, consumers select the effective one using standard label semantics (e.g., newest `cts`, respecting `neg`/`exp` where applicable).
  * **Annotation record schema constraint:** any record type used as an annotation must include a `target` field containing the annotated subject URI (an `at://` URI) that matches the label `uri`.
  * **Authorship / trust constraint:** the annotation record referenced by `ref` must be authored by the labeler DID in `src` (i.e., the record’s repo DID equals `label.src`).

    * This avoids needing a strong ref (`refCid`) while ensuring the consumer’s trust in the labeler extends to the referenced record.
* App Views can include annotations from subscribed labelers when hydrating posts (similar to embeds).
* App Views MAY return multiple annotations; App View or client policy determines selection and presentation.

---

## Annotation Types

A note adding context to a post is one possible type of annotation. Others might include alt text, citations, and highlights. Clients will need to know how to display annotations depending on the annotation type.

---

## Compatibility with W3C Annotations Protocol

This model is very compatible with the W3C Web Annotation Data Model and Protocol. In the W3C model, an Annotation has a `target` (e.g., a URL or URI) and a `body` (the annotation content, tag, or linked resource).

It would be straightforward for an App View to act as a W3C-compatible [Annotations Server](https://www.w3.org/TR/annotation-protocol/#annotation-retrieval) for interop, serving JSON-LD annotations whose `target` is the annotated URL/URI and whose `body` either embeds the annotation content or identifies the referenced atproto annotation record.

---

## Pros

* User choice of annotators; apps can have default annotators; leverages labeler infrastructure.
* Efficient batching/hydration via App Views reduces client-side work.
* Optional W3C Annotations interop.

## Cons

* Requires formal lexicon updates (e.g., label defs) and careful rollout for strict validators.

---

## Worked Example

### Label Record

```json
{
  "src": "did:plc:57fl6zy4wmpuknwpgtjqkvlz",
  "uri": "at://did:plc:ywbm3iywnhzep3ckt6efhoh7/app.bsky.feed.post/3m2mryxy7zc2c",
  "val": "helpful-community-note",
  "ref": "at://did:plc:57fl6zy4wmpuknwpgtjqkvlz/org.opencommunitynotes.note/3m2mryxy7zc2c-note-hash123",
  "cts": "2025-12-27T14:46:48.830Z"
}
```

**Consumer checks:**

* The `ref` record’s repo DID equals `src`.
* The referenced record includes `target == uri`.

### Annotation Record (Community Note)

```json
{
  "$type": "org.opencommunitynotes.note",
  "target": "at://did:plc:ywbm3iywnhzep3ckt6efhoh7/app.bsky.feed.post/3m2mryxy7zc2c",
  "text": "This claim is outdated; the event occurred in 2024, not 2025. See https://source.com/article",
  "reasons": ["outdated_information"],
  "author-pseudonym": "mellow-jazzy-tunes",
  "createdAt": "2025-12-12T12:00:00Z"
}
```

### App View Hydrated Post

```json
{
  "...post data...": "...",
  "annotations": [
    {
      "$type": "app.bsky.annotation",
      "record": {
        "uri": "at://did:plc:57fl6zy4wmpuknwpgtjqkvlz/org.opencommunitynotes.note/3m2mryxy7zc2c-note-hash123",
        "cid": "bafyre...example",
        "value": {
          "$type": "org.opencommunitynotes.note",
          "target": "at://did:plc:ywbm3iywnhzep3ckt6efhoh7/app.bsky.feed.post/3m2mryxy7zc2c",
          "text": "This claim is outdated; the event occurred in 2024, not 2025. See https://source.com/article",
          "reasons": ["outdated_information"],
          "author-pseudonym": "mellow-jazzy-tunes",
          "createdAt": "2025-12-12T12:00:00Z",
          "score": 0.8
        },
        "url": "https://bluenotes.social/notes/3m2mryxy7zc2c-note-hash123"
      }
    }
  ]
}
```

### Optional W3C Annotation Retrieval (Interop)

An atproto label maps cleanly to a W3C annotation. For example, this atproto label:

```json
{
  "src": "did:plc:labelerexample123",
  "uri": "at://did:plc:alice/app.bsky.feed.post/3kxyz123",
  "val": "spam",
  "cts": "2026-03-27T15:42:11Z"
}
```

Might map to a W3C annotation that looks like this:

```json
{
  "@context": "http://www.w3.org/ns/anno.jsonld",
  "type": "Annotation",
  "motivation": "classifying",
  "target": "at://did:plc:alice/app.bsky.feed.post/3kxyz123",
  "body": {
    "type": "SemanticTag",
    "id": "at://did:plc:labelerexample/app.bsky.labeler.service/self#spam"
  }
}
```

Here the fragment (`#spam`) is just an example convention for identifying a specific label definition within the labeler service record.

Any labeler/annotator could implement the W3C Annotations Protocol's REST API for retrieving annotations. When labels include an additional `ref` field pointing to annotation content, that content can be embedded in the W3C annotation body:

**Request**

```http
GET /annotations/anno1 HTTP/1.1
Host: bluenotes.social
Accept: application/ld+json; profile="http://www.w3.org/ns/anno.jsonld"
```

**Response**

```http
HTTP/1.1 200 OK
Content-Type: application/ld+json; profile="http://www.w3.org/ns/anno.jsonld"
Link: <http://www.w3.org/ns/ldp#Resource>; rel="type"
Vary: Accept
```

```json
{
  "@context": "http://www.w3.org/ns/anno.jsonld",
  "id": "https://bluenotes.social/annotations/anno1",
  "type": "Annotation",
  "motivation": "commenting",
  "created": "2025-12-12T12:00:00Z",
  "body": {
    "type": "TextualBody",
    "value": "This claim is outdated; the event occurred in 2024, not 2025. See https://source.com/article"
  },
  "target": "at://did:plc:ywbm3iywnhzep3ckt6efhoh7/app.bsky.feed.post/3m2mryxy7zc2c"
}
```
