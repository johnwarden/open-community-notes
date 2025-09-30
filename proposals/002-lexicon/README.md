# Proposal: Open Community Notes Lexicon

Open Community Notes uses the [social.pmsky lexicon](https://docs.pmsky.social/tech/lexicon). The Drew McArthur, the maintainer of (pmsky)[https://pmsky.social/) lexicon has kindly expanded the lexicon to meet our requirements.

- `social.pmsky.proposal`: For proposing a moderation action (e.g. adding a label w/ note to a post).
- `social.pmsky.vote`: For approving or disapproving of proposals.

## 2. Proposal Record (`social.pmsky.proposal`)

### 2.1. Spec

[https://github.com/pmsky-social/app/blob/main/lexicons/proposal.json](https://github.com/pmsky-social/app/blob/main/lexicons/proposal.json)

| Field Name | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `ver` | `integer` | No | The AT Protocol version of the proposal object. |
| `typ` | `string` | Yes | The type of moderation action being proposed. e.g. 'label'. |
| `src` | `string` (did) | Yes | DID of the actor who created this proposal. |
| `uri` | `string` (uri) | Yes | AT URI of the resource that this proposal applies to. |
| `cid` | `string` (cid) | No | Optionally, CID specifying the specific version of 'uri' resource. |
| `val` | `string` | Yes | The short string name of the value of the proposed label. |
| `note` | `string` | No | The full text of an proposed annotation. |
| `reasons`| `string[]` | No | An optional array of predefined reasons justifying the moderation action. |
| `aid` | `string` | No | The persistent, anonymous identifier for the user creating the proposal. |
| `cts` | `string` (datetime)| Yes | Timestamp when this proposal was created. |
| `sig` | `bytes` | No | Signature of dag-cbor encoded proposal. |

### 2.2. Defined Values for `reasons`

| Value |
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
  "$type": "social.pmsky.proposal",
  "typ": "label",
  "src": "did:plc:communitynotesservice",
  "uri": "at://did:plc:xxxxxxxxxxxx/app.bsky.feed.post/3kabc123xyz",
  "val": "note",
  "note": "This photo was taken in 2015, not during the recent events.",
  "reasons": ["outdated_information"],
  "aid": "anon:ab34fec9de56",
  "cts": "2025-06-25T18:00:00.000Z"
}
```

## 3. Vote Record (`social.pmsky.vote`)

[https://github.com/pmsky-social/app/blob/main/lexicons/vote.json](https://github.com/pmsky-social/app/blob/main/lexicons/vote.json)


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

| Value |
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
| `helpful_other` |
| `not_helpful_other` |

### 3.3. Example JSON
```json
{
  "$type": "social.pmsky.vote",
  "src": "did:plc:somevoter",
  "uri": "at://did:plc:communitynotesservice/social.pmsky.proposal/3kprop123abc",
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

## 4. Dispute Mechanism: Proposing a Label on a Label

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
  "$type": "social.pmsky.proposal",
  "typ": "label",
  "src": "did:plc:communitynotesservice",
  "uri": "at://did:plc:communitynotesservice/social.pmsky.proposal/3klabel987zyx",
  "val": "label-incorrect",
  "note": "The original note claims the photo is from 2015, but it's actually current.Here's the source.",
  "aid": "anon:userB",
  "cts": "2025-06-27T09:00:00.000Z"
}
```


## 5. Design Decisions and Rationale

* **Lexicon Compatibility:** This lexicon is a superset of the social.pmsky lexicon to maximize potential interoperability. And in that lexicon, the `label` record identical to `com.atproto.label.defs#label`but as a concrete record type. This allows standard clients to process these records by ignoring the extra fields (`reason`, `note`, `aid`).
* **Generality:** Works not just for Notes, and not just for labels, but for moderation actions in general.
* **Optional Note Field:** A label with a note field is a note. Some types of labels can require a note, others don'.t
* **Flexible Subject Fields:** The `proposal` record uses top-level `uri` and optional `cid` fields to target any content on the web. The `rating` record uses the standard `com.atproto.repo.strongRef` `subject` field because it *must* target a `label` record, which always exists within the AT Protocol. Support for external content is achieved by making the top-level `cid` field in the `label` record optional.
* **Recursive Dispute Mechanism:** The protocol supports disputes by reusing the `label` record to target proposals. This elegant, recursive design keeps the lexicon simple while enabling complex moderation dialogues.
* **Precedent from Existing Systems:** The values for `reason` and the `reasons` for ratings are adapted from X's Community Notes to build on a proven model.
* **Embedded Sources:** To allow for multiple references, all source links are embedded directly within the optional `note` field.
* **Anonymity:** The `aid` field allows a single service account to sign all records but include anonymous IDs so that the Community Notes algorithm can be run transparently
