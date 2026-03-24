# ATProto Annotations Proposal

## Overview

* Any labeler can also act as an **annotator**.
* An annotator publishes annotations by adding an optional `ref` field to label records.
* `ref` contains a URI (weak ref) of an atproto record of any type.
    * **Coherence/supersession:** for a given labeler `src`, the tuple `(uri, val)` has a single coherent annotation reference at a time (i.e., one current `ref`). When multiple label records exist for the same `(src, uri, val)`, consumers select the effective one using standard label semantics (e.g., newest `cts`, respecting `neg`/`exp` where applicable).
    * **Annotation record schema constraint:** any record type used as an annotation must include a `target` field containing the annotated subject URI (an `at://` URI) that matches the label `uri`.
    * **Authorship / trust constraint:** the annotation record referenced by `ref` must be authored by the labeler DID in `src` (i.e., the record’s repo DID equals `label.src`).
        * This avoids needing a strong ref (`refCid`) while ensuring the consumer’s trust in the labeler extends to the referenced record.
* App Views can include annotations from subscribed labelers when hydrating posts. Annotations can go into the existing `embed` field, using `app.bsky.embed.record` (or a dedicated embed type later, if desired).

---

## Presentation for Lexicon Embeds

This proposal requires a general-purpose solution for [lexicon embeds](https://discourse.atprotocol.community/t/lexicon-embeds-overview/133/5): allowing apps to display embedded records from any lexicon.

I propose a [web config](https://discourse.atprotocol.community/t/lexicon-embeds-overview/133/5?u=jonathanwarden.com) record that defines URLs for obtaining presentation data for a record type (oEmbed/Open Graph/Open Frames/Block Kit/Web Tiles/etc.). In practice, clients would usually obtain presentation payloads via App View hydration, but the web config enables discovery and fallback behavior.

App Views that hydrate posts with annotations should also return presentation data for the annotation record type, so clients can render it without pre-knowledge of the schema:

* For **oEmbed**, hydration returns the **oEmbed endpoint** and the canonical **URL** for the annotation record; hydration does **not** return HTML.
* For **Open Graph / Open Frames / Block Kit-like** schemas, hydration returns the **actual structured presentation payload** (safe to transport; rendering policy remains client-defined).

---

## Compatibility with W3C Annotations protocol

This model is very compatible with the W3C Web Annotation Data Model and Protocol. In the W3C model the Annotation has a `target` (e.g., URL) and a `body` (the annotation content).

It would be straightforward for an App View to act as a W3C-compatible [Annotations Server](https://www.w3.org/TR/annotation-protocol/#annotation-retrieval) for interop, serving JSON-LD annotations whose `target` is the annotated URL/URI and whose `body` embeds (or references) the atproto annotation record.

---

## Pros

* User choice of annotators; apps can have default annotators; leverages labeler infrastructure.
* Efficient batching/hydration via App Views reduces client-side work.
* Optional interop path with non-atproto apps via W3C REST without making web presentation part of the core protocol flow.
* Optional W3C Annotations interop.

## Cons

* Requires formal lexicon updates (e.g., label defs) and careful rollout for strict validators.
* If App Views don’t hydrate, client-side calls to fetch presentation payloads add latency/duplication.

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

### Annotations Record (Community Note)

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

### Lexicon Web Config (Supported Presentation Formats)

Lexicon: `com.atproto.lexicon.web` (stored alongside `com.atproto.lexicon.schema` for `org.opencommunitynotes.note`)

```json
{
  "lexiconId": "org.opencommunitynotes.note",
  "urlTemplate": "https://bluenotes.social/notes/{rkey}",
  "presentation": {
    "oembed": {
      "endpoint": "https://bluenotes.social/oembed"
    },
    "blockkit": {
      "endpoint": "https://bluenotes.social/blockkit"
    },
    "opengraph": true,
    "openframes": true
  }
}
```

Notes:

* `oembed` and `blockkit` expose explicit endpoints.
* `opengraph` and `openframes` indicate that tags can be obtained from the web URL constructed via `urlTemplate`.

### App View Hydrated Post (Record + Presentation)

```json
{
  "...post data...": "...",
  "embed": {
    "$type": "app.bsky.embed.record",
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
      }
    },
    "url": "https://bluenotes.social/notes/3m2mryxy7zc2c-note-hash123",
    "presentation": {
      "oembed": {
        "endpoint": "https://bluenotes.social/oembed"
      },
      "opengraph": { "...content...": "..." },
      "openframes": { "...content...": "..." },
      "blockkit": { "...content...": "..." }
    }
  }
}
```

### Optional W3C Annotation Retrieval (Interop)

Example (modeled on the W3C Protocol’s retrieval pattern):

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
  "created": "2025-12-12T12:00:00Z",
  "body": {
    "id": "at://did:plc:57fl6zy4wmpuknwpgtjqkvlz/org.opencommunitynotes.note/3m2mryxy7zc2c-note-hash123",
    "type": "TextualBody",
    "value": "This claim is outdated; the event occurred in 2024, not 2025. See https://source.com/article"
  },
  "target": "at://did:plc:ywbm3iywnhzep3ckt6efhoh7/app.bsky.feed.post/3m2mryxy7zc2c"
}
```


