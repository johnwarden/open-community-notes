# Proposal: Open Community Notes Lexicon

## 1. Introduction

This document proposes a new lexicon for the Open Community Notes per the main [Open Community Notes Architecture proposal](https://github.com/johnwarden/open-community-notes/tree/master/proposals/001-architecture).

Open Community Notes is mean to be a flexible, generalized community-driven moderation system capable of handling everything from adding context to flagging spam and abuse. Crucially, it is designed to target content both within the AT Protocol and on the wider web (e.g., news articles, posts on other social networks).

## 2. Reusable Definitions (`org.opencommunitynotes.defs`)

### 2.1. `subjectRef`

This is a custom reference object that can point to any piece of content, on or off the AT Protocol. It supports both strong, verifiable references and weak, location-only references.

| Field Name | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `uri` | `string` (format: `uri`) | Yes | The URI of the content being targeted (e.g., `at://...` or `https://...`). |
| `cid` | `string` | No | The Content Identifier (CID) of the subject, if it is an AT-Proto record. Its presence indicates a strong reference. |

## 3. Label Record (`org.opencommunitynotes.label`)

This record represents a single label applied to a piece of content by a contributor.

### 3.1. Spec

| Field Name | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `subject` | `org.opencommunitynotes.defs#subjectRef`| Yes | A reference to the content being labeled. |
| `label` | `string` | Yes | A string identifying the label being applied, using a dot-notation hierarchy. |
| `text` | `string` | No | An optional explanation. Required for parametric labels (e.g., `context.*`). Source links are embedded here. |
| `contributorId` | `string` | Yes | The persistent, anonymous identifier for the user applying the label. |
| `createdAt` | `string` (format: `datetime`)| Yes | The timestamp of when the label was applied. |

### 3.2. Example Label Values

* **Contextual Labels (Parametric - require `text`)**: `context.factual_error`, `context.misrepresentation_or_missing_context`, `context.altered_media`, etc.
* **Flagging Labels (Non-Parametric - `text` is optional)**: `spam`, `abuse.harassment`, `abuse.threat_of_violence`, etc.

### 3.3. Example Note Record JSON

**Example of a Strong Reference to an AT-Proto Post:**
```json
{
  "$type": "org.opencommunitynotes.label",
  "subject": {
    "uri": "at://did:plc:xxxxxxxxxxxx/app.bsky.feed.post/3kabc123xyz",
    "cid": "bafyreibxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  },
  "label": "context.factual_error",
  "text": "This post contains an incorrect statistic. The actual number is 42%, not 52%. Source: [https://example.com/stats](https://example.com/stats)",
  "contributorId": "anon:ab34fec9de56",
  "createdAt": "2025-06-25T19:00:00.000Z"
}
```

**Example of a Weak Reference to a Mastodon Post:**
```json
{
  "$type": "org.opencommunitynotes.label",
  "subject": {
    "uri": "[https://mastodon.social/@user/123456789](https://mastodon.social/@user/123456789)"
  },
  "label": "spam",
  "contributorId": "anon:cd56ef78ab90",
  "createdAt": "2025-06-25T19:05:00.000Z"
}
```

## 4. Vote Record (`org.opencommunitynotes.vote`)

This record represents a user's rating on the helpfulness or validity of a specific label.

### 4.1. Spec

| Field Name | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `subject` | `com.atproto.repo.strongRef`| Yes | A **strong reference** to the `org.opencommunitynotes.label` record being voted on. |
| `helpfulness` | `string` | Yes | The rating. Must be one of `"helpful"`, `"somewhat_helpful"`, or `"not_helpful"`. |
| `reasons` | `string[]` | No | An optional array of predefined reasons justifying the rating. |
| `contributorId` | `string` | Yes | The persistent, anonymous identifier for the user casting the vote. |
| `createdAt` | `string` (format: `datetime`)| Yes | The timestamp of when the vote was cast. |


### 4.2. Defined Values for `reasons`

The list of valid reasons is derived from the user-facing UI options, with lexicon keys aligned to X's Community Notes ratings data model for consistency.

**Valid reasons if `helpfulness` is `"helpful"` or `"somewhat_helpful"`:**

| Lexicon Key | UI Text / Aligns with `helpful*` | 
 | ----- | ----- | 
| `cites_good_sources` | Cites high-quality sources | 
| `is_clear` | Easy to understand | 
| `addresses_claim` | Directly addresses the post's claim | 
| `provides_important_context` | Provides important context | 
| `is_unbiased` | Neutral or unbiased language | 
| `other` | Other | 

**Valid reasons if `helpfulness` is `"not_helpful"`:**

| Lexicon Key | UI Text / Aligns with `notHelpful*` | 
 | ----- | ----- | 
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

### 4.3. Example JSON

```json
{
  "$type": "org.opencommunitynotes.vote",
  "subject": {
    "uri": "at://did:plc:pds_host_did/org.opencommunitynotes.label/3klabelrecordkey",
    "cid": "bafyreidddddddddddddddddddddddddddddddddddd"
  },
  "helpfulness": "not_helpful",
  "reasons": [
    "is_opinion_or_speculation",
    "sources_dont_support_note"
  ],
  "contributorId": "anon:ef78ab90cd56",
  "createdAt": "2025-06-25T18:10:00.000Z"
}
```

## 5. Design Decisions and Rationale

* **Universal Subject References:** The lexicon supports labeling content anywhere on the web via a custom `subjectRef` object containing a `uri` and an optional `cid`. The presence of a `cid` indicates a "strong" (verifiable) reference to an AT-Proto object, while its absence indicates a "weak" reference to external content.
* **Generalized Labeling Protocol:** The system is designed as a flexible labeling protocol. This allows the community to apply a wide range of hierarchical labels (e.g., `context.*`, `spam`, `abuse.*`) using the same core records.
* **Precedent from Existing Systems:** To build on a proven model, the initial set of `context.*` labels and the `reasons` for voting are directly adapted from the user-facing options and data model of X's Community Notes. The list of reasons prioritizes the current UI, excluding legacy fields from the data model, to create a clean and relevant specification.
* **Parametric vs. Non-Parametric Labels:** The design distinguishes between labels that require additional data (like `text` for a `context` note) and those that are self-sufficient (like a `spam` flag) by making the `text` field optional.
* **Embedded Sources:** To align with established patterns and allow for multiple references, all source links are embedded directly within the optional `text` field.
* **Contextual Voting:** The `vote` record is used to assess the quality of an applied `label`. Client applications can adapt the UI, for instance by hiding the specific `reasons` checkboxes when voting on simple flags where they may not be relevant.
* **Lexicon Namespace & Anonymity:** The `org.opencommunitynotes` namespace reflects the project's independent nature, and the `contributorId` field enables a reputation system built on persistent, anonymous identifiers.

