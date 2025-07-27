# Proposal: Open Community Notes Lexicon

## 1. Introduction

This document proposes a new lexicon for the Open Community Notes per the main [Open Community Notes Architecture proposal](https://github.com/johnwarden/open-community-notes/tree/master/proposals/001-architecture).

This lexicon defines three main record types:

- `org.opencommunitynotes.proposal`: For proposing a moderation action (adding a label or annotation to a post).
- `org.opencommunitynotes.vote`: For approving or disapproving of proposals.
- `org.opencommunitynotes.label`: Like `com.atproto.label.defs#label`, but as a concrete record type, and with additional optional fields 'note' and 'proposal'.


These are supersets of the `social.pmsky.proposal`, `social.pmsky.vote`, and `social.pmsky.label` lexicons.

## 2. Proposal Record (`org.opencommunitynotes.proposal`)

A proposed moderation action (e.g. adding a label or annotation to a post). Refers to some other resource via URI (e.g. an atproto post).

### 2.1. Spec

| Field Name | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `ver` | `integer` | No | The AT Protocol version of the proposal object. |
| `typ` | `string` | Yes | The type of moderation action being proposed. e.g. 'post_label'. |
| `src` | `string` (did) | Yes | DID of the actor who created this proposal. |
| `uri` | `string` (uri) | Yes | AT URI of the resource that this proposal applies to. |
| `cid` | `string` (cid) | No | Optionally, CID specifying the specific version of 'uri' resource. |
| `val` | `string` | Yes | The short string name of the value of the proposed label. |
| `note` | `string` | No | For 'readers-added-context' labels, the full text of the proposed annotation. |
| `reasons`| `string[]` | No | An optional array of predefined reasons justifying the moderation action. |
| `aid` | `string` | No | The persistent, anonymous identifier for the user creating the proposal. |
| `cts` | `string` (datetime)| Yes | Timestamp when this proposal was created. |
| `exp` | `string` (datetime)| No | Timestamp at which this proposal expires. |
| `sig` | `bytes` | No | Signature of dag-cbor encoded proposal. |

### 2.2. Defined Values for `reasons`

| Lexicon Key |
| :--- |
| `factual_error` |
| `altered_media` |
| `outdated_information` |
| `misrepresentation_or_missing_context` |
| `unverified_claim_as_fact` |
| `joke_or_satire` |
| `other` |

### 2.3. Example JSON
```json
{
  "$type": "org.opencommunitynotes.proposal",
  "typ": "post_label",
  "src": "did:plc:communitynotesservice",
  "uri": "at://did:plc:xxxxxxxxxxxx/app.bsky.feed.post/3kabc123xyz",
  "val": "readers-added-context",
  "note": "This photo was taken in 2015, not during the recent events.",
  "reasons": ["outdated_information"],
  "aid": "anon:ab34fec9de56",
  "cts": "2025-06-25T18:00:00.000Z"
}
```

## 3. Vote Record (`org.opencommunitynotes.vote`)

A vote record, representing a user's approval or disapproval of the referenced resource (e.g. voting for/against a proposal, upvoting/downvoting a post).

### 3.1. Spec

| Field Name | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `src` | `string` (did) | Yes | The account creating the vote. |
| `uri` | `string` (uri) | Yes | AT URI of the resource that this vote applies to. |
| `cid` | `string` (cid) | No | Optionally, CID specifying the specific version of 'uri' resource. |
| `val` | `integer` | Yes | +1 for 'approval', -1 for 'disapproval', 0 for 'neutrality'. |
| `reasons`| `string[]` | No | An optional array of predefined reasons justifying the rating. |
| `aid` | `string` | No | The persistent, anonymous identifier for the user casting the vote. |
| `cts` | `string` (datetime)| Yes | Timestamp when this vote was created. |
| `sig` | `bytes` | No | Signature of dag-cbor encoded vote. |

### 3.2. Defined Values for `reasons`

| Lexicon Key |
| :--- |
| `cites_high_quality_sources` |
| `is_clear` |
| `addresses_claim` |
| `provides_important_context` |
| `is_unbiased` |
| `sources_missing_or_unreliable` |
| `sources_dont_support_note` |
| `is_incorrect` |
| `is_opinion_or_speculation` |
| `is_hard_to_understand` |
| `is_off_topic_or_irrelevant` |
| `is_argumentative_or_biased` |
| `note_not_needed` |
| `is_spam_harassment_or_abuse` |
| `other` |

### 3.3. Example JSON
```json
{
  "$type": "org.opencommunitynotes.vote",
  "src": "did:plc:somevoter",
  "uri": "at://did:plc:communitynotesservice/org.opencommunitynotes.proposal/3kprop123abc",
  "cid": "bafyreidddddddddddddddddddddddddddddddddddd",
  "val": 1,
  "reasons": [
    "cites_high_quality_sources",
    "is_clear"
  ],
  "aid": "anon:ef78ab90cd56",
  "cts": "2025-06-25T18:10:00.000Z"
}
```

## 4. Label Record (`org.opencommunitynotes.label`)

Like `com.atproto.label.defs#label`, but as a concrete record type, and with additional optional fields 'note' and 'proposal'.

### 4.1. Spec

| Field Name | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `ver` | `integer` | No | The AT Protocol version of the label object. |
| `src` | `string` (did) | Yes | DID of the actor who created this label. |
| `uri` | `string` (uri) | Yes | AT URI of the resource that this label applies to. |
| `cid` | `string` (cid) | No | Optionally, CID specifying the specific version of 'uri' resource. |
| `val` | `string` | Yes | The short string name of the value or type of this label. |
| `note` | `string` | No | The full text of any annotation associated with this label. |
| `proposal`| `ref` | No | A strong reference to the proposal that created this label. |
| `neg` | `boolean` | No | If true, this is a negation label, removing a previous label. |
| `cts` | `string` (datetime)| Yes | Timestamp when this label was created. |
| `exp` | `string` (datetime)| No | Timestamp at which this label expires. |
| `sig` | `bytes` | No | Signature of dag-cbor encoded label. |

### 4.2. Example JSON
```json
{
  "$type": "org.opencommunitynotes.label",
  "src": "did:plc:communitynotesservice",
  "uri": "at://did:plc:xxxxxxxxxxxx/app.bsky.feed.post/3kabc123xyz",
  "cid": "bafyreibxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "val": "readers-added-context",
  "note": "This photo was taken in 2015, not during the recent events.",
  "proposal": {
      "uri": "at://did:plc:communitynotesservice/org.opencommunitynotes.proposal/3kprop123abc",
      "cid": "bafyreidddddddddddddddddddddddddddddddddddd"
  },
  "cts": "2025-06-26T10:00:00.000Z"
}
```

## 5. Dispute Mechanism: Proposing a Label on a Label

A crucial feature of a robust moderation system is the ability to dispute or rebut a label. This protocol handles disputes by reusing the `proposal` records to create a "meta-label" that targets another proposal. This avoids adding new record types to the lexicon, keeping the protocol lean.

The mechanism is as follows:
1. A user creates a new `proposal` record.
2. The `uri` of this new proposal points to the original `proposal` being disputed.
3. The `val` of this proposal could be `disputed` or `incorrect`, and the `note` field is used to explain why the original label is incorrect, biased, or unnecessary.
4. If the dispute proposal passes:
  - if the proposal being disputed was approved and a `label` record published, a negation of that label is published (a copy of the label record with `neg` field set to true)
  - there is no need to "label a label" as "incorrect" -- simply negating it is sufficient.

This creates a chain of context that can be algorithmically scored and presented to users, allowing for a richer, more transparent moderation process.

### 5.1. Example Dispute Proposal JSON
```json
{
  "$type": "org.opencommunitynotes.proposal",
  "typ": "post_label",
  "src": "did:plc:communitynotesservice",
  "uri": "at://did:plc:communitynotesservice/org.opencommunitynotes.proposal/3klabel987zyx",
  "val": "label-incorrect",
  "note": "The original note claims the photo is from 2015, but it's actually current.Here's the source.",
  "aid": "anon:userB",
  "cts": "2025-06-27T09:00:00.000Z"
}
```


## 6. Design Decisions and Rationale

* **Lexicon Compatibility:** This lexicon is a superset of the social.pmsky lexicon to maximize potential interoperability. And in that lexicon, the `label` record identical to `com.atproto.label.defs#label`but as a concrete record type. This allows standard clients to process these records by ignoring the extra fields (`reason`, `note`, `aid`).
* **Flexible Subject Fields:** The `label` record uses top-level `uri` and optional `cid` fields to target any content on the web. The `rating` record uses the standard `com.atproto.repo.strongRef` `subject` field because it *must* target a `label` record, which always exists within the AT Protocol. Support for external content is achieved by making the top-level `cid` field in the `label` record optional.
* **Recursive Dispute Mechanism:** The protocol supports disputes by reusing the `label` record to target propsals. This elegant, recursive design keeps the lexicon simple while enabling complex moderation dialogues.
* **Precedent from Existing Systems:** The values for `reason` (when `val` is `"needs-context"`) and the `reasons` for ratings are adapted from X's Community Notes to build on a proven model.
* **Parametric vs. Non-Parametric Labels:** The design distinguishes between labels that require `note` (like `needs-context` labels) and those that don't (like `spam`) by keeping the `note` field optional.
* **Embedded Sources:** To allow for multiple references, all source links are embedded directly within the optional `note` field.
* **Anonymity:** The `aid` field allows a single service account to sign all records but include anonymous IDs so that the Community Notes algorithm can be run transparently
