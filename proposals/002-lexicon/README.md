# Proposal: Open Community Notes Lexicon

## 1. Introduction

This document proposes a new lexicon for the Open Community Notes per the main [Open Community Notes Architecture proposal](https://github.com/johnwarden/open-community-notes/tree/master/proposals/001-architecture).

Open Community Notes is mean to be a flexible, generalized community-driven moderation system capable of handling everything from adding context to flagging spam and abuse. Crucially, it is designed to target content both within the AT Protocol and on the wider web (e.g., news articles, posts on other social networks).

## 2. Label Record (`org.opencommunitynotes.label`)

This record represents a single label applied to a piece of content. It extends com.atproto.label AT-Proto with fields necessary for community notes.

### 2.1. Spec

| Field Name | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `src` | `string` (did) | Yes | The DID of the service applying the label. For this protocol, it will always be the DID of the Community Notes service. |
| `uri` | `string` (uri) | Yes | The URI of the content being labeled (e.g., `at://...` or `https://...`). |
| `cid` | `string` | No | The CID of the subject if it is an AT-Proto record. Its presence indicates a strong reference. |
| `val` | `string` | Yes | The primary label string (e.g., `readers-added-context`). |
| `reason` | `string` | No | An optional, more specific reason for the primary label. |
| `text` | `string` | No | An optional explanation. Required for parametric labels like `readers-added-context`. Source links are embedded here. |
| `anonId` | `string` | Yes | The persistent, anonymous identifier for the user applying the label. |
| `cts` | `string` (datetime)| Yes | The timestamp of when the label was applied. |

### 2.2. Defined Values

This lexicon proposes a new primary label (`val`) for adding context to a post: `readers-added-context`.

When `val` is `readers-added-context`, the `reason` field should be used to provide a more specific classification. The defined values for `reason` in this context are:

| `reason` Key | UI Text |
| :--- | :--- |
| `factual_error` | It contains a factual error |
| `altered_media` | It contains a digitally altered photo or video |
| `outdated_information` | It contains outdated information that may be misleading |
| `misrepresentation_or_missing_context` | It is a misrepresentation or missing important context |
| `unverified_claim_as_fact` | It presents an unverified claim as a fact |
| `joke_or_satire` | It is a joke or satire that might be misinterpreted as a fact |
| `other` | Other |

### 2.3. Example JSON

**Example of a `readers-added-context` Label on an AT-Proto Post:**
```json
{
  "$type": "org.opencommunitynotes.label",
  "src": "did:plc:communitynotesservice",
  "uri": "at://did:plc:xxxxxxxxxxxx/app.bsky.feed.post/3kabc123xyz",
  "cid": "bafyreibxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "val": "readers-added-context",
  "reason": "factual_error",
  "text": "This post contains an incorrect statistic. The actual number is 42%, not 52%. Source: [https://example.com/stats](https://example.com/stats)",
  "anonId": "anon:ab34fec9de56",
  "cts": "2025-06-25T19:00:00.000Z"
}
```

## 3. Rating Record (`org.opencommunitynotes.rating`)

This record represents a user's rating on the helpfulness or validity of a specific label.

### 3.1. Spec

| Field Name | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `subject` | `com.atproto.repo.strongRef`| Yes | A **strong reference** to the `org.opencommunitynotes.label` record being rated. |
| `val` | `string` | Yes | The rating value. Must be one of `"helpful"`, `"somewhat_helpful"`, or `"not_helpful"`. |
| `reasons` | `string[]` | No | An optional array of predefined reasons justifying the rating. |
| `anonId` | `string` | Yes | The persistent, anonymous identifier for the user casting the rating. |
| `createdAt` | `string` (datetime)| Yes | The timestamp of when the rating was cast. |

### 3.2. Defined Values for `reasons`

The list of valid reasons is derived from the user-facing UI options, with lexicon keys aligned to X's Community Notes ratings data model for consistency.

**Valid reasons if `val` is `"helpful"` or `"somewhat_helpful"`:**

| Lexicon Key | UI Text / Aligns with `helpful*` |
| :--- | :--- |
| `cites_high_quality_sources` | Cites high-quality sources |
| `is_clear` | Easy to understand |
| `addresses_claim` | Directly addresses the post's claim |
| `provides_important_context` | Provides important context |
| `is_unbiased` | Neutral or unbiased language |
| `other` | Other |

**Valid reasons if `val` is `"not_helpful"`:**

| Lexicon Key | UI Text / Aligns with `notHelpful*` |
| :--- | :--- |
| `sources_missing_or_unreliable` | Sources not included or unreliable |
| `sources_dont_support_note` | Sources do not support note |
| `is_incorrect` | Incorrect information |
| `is_opinion_or_speculation` | Opinion or speculation |
| `is_hard_to_understand` | Typos or unclear language |
| `is_off_topic_or_irrelevant` | Misses key points or irrelevant |
| `is_argumentative_or_biased` | Argumentative or biased language |
| `note_not_needed` | Note not needed on this post |
| `is_spam_harassment_or_abuse` | Spam, harassment, or abuse |
| `other` | Other |

### 3.3. Example JSON
```json
{
  "$type": "org.opencommunitynotes.rating",
  "subject": {
    "uri": "at://did:plc:pds_host_did/org.opencommunitynotes.label/3klabelrecordkey",
    "cid": "bafyreidddddddddddddddddddddddddddddddddddd"
  },
  "val": "not_helpful",
  "reasons": [
    "is_opinion_or_speculation",
    "sources_dont_support_note"
  ],
  "anonId": "anon:ef78ab90cd56",
  "createdAt": "2025-06-25T18:10:00.000Z"
}
```

## 4. Dispute Mechanism: Labeling a Label

A crucial feature of a robust moderation system is the ability to dispute or rebut a label. This protocol handles disputes by reusing the `org.opencommunitynotes.label` record itself, creating a "meta-label" that targets another label. This avoids adding a new record type to the lexicon, keeping the protocol lean.

The mechanism is as follows:
1. A user creates a new `label` record.
2. The `uri` and `cid` of this new label point to the original `label` being disputed.
3. The `val` of this new label is set to `"label-not-needed"`.
4. The `text` field is used to explain why the original label is incorrect, biased, or unnecessary.
5. The `reason` field can optionally be used to add further classification to the dispute, using the same keys defined for "not helpful" ratings (e.g., `is_incorrect`, `is_argumentative_or_biased`).

This creates a chain of context that can be algorithmically scored and presented to users, allowing for a richer, more transparent moderation process.

### 4.1. Example Dispute JSON
```json
{
  "$type": "org.opencommunitynotes.label",
  "src": "did:plc:communitynotesservice",
  "uri": "at://did:plc:pds_host_did/org.opencommunitynotes.label/original_label_key",
  "cid": "bafyreicccccccccccccccccccccccccccccccc",
  "val": "label-not-needed",
  "reason": "is_opinion_or_speculation",
  "text": "The context added in the original note is not a factual correction but rather the author's personal opinion on the matter.",
  "anonId": "anon:userB",
  "createdAt": "2025-06-25T20:00:00.000Z"
}
```

## 5. Design Decisions and Rationale

* **Superset Compatibility:** The `label` record is a functional superset of `com.atproto.label.defs#label`. This allows standard clients to process these records by ignoring the extra fields (`reason`, `text`, `anonId`).
* **Divergent Subject Fields:** The `label` record uses top-level `uri` and optional `cid` fields to target any content on the web. The `rating` record uses the standard `com.atproto.repo.strongRef` `subject` field because it *must* target a `label` record, which always exists within the AT Protocol. This distinction is intentional and provides maximum flexibility for labels while ensuring strict verifiability for ratings.
* **Universal Subject References:** Support for external content is achieved by making the top-level `cid` field in the `label` record optional. Its presence signifies a strong, verifiable reference to AT-Proto content, while its absence implies a weak reference to an external URI.
* **Recursive Dispute Mechanism:** The protocol supports disputes by reusing the `label` record to target other labels. This elegant, recursive design keeps the lexicon simple while enabling complex moderation dialogues.
* **Precedent from Existing Systems:** The values for `reason` (when `val` is `"readers-added-context"`) and the `reasons` for ratings are adapted from X's Community Notes to build on a proven model.
* **Parametric vs. Non-Parametric Labels:** The design distinguishes between labels that require `text` (like `"readers-added-context"`) and those that don't (like `"spam"`) by keeping the `text` field optional.
* **Embedded Sources:** To allow for multiple references, all source links are embedded directly within the optional `text` field.
* **Lexicon Namespace & Anonymity:** The `org.opencommunitynotes` namespace reflects the project's independent nature, and the `anonId` field enables a reputation system built on persistent, anonymous identifiers.

