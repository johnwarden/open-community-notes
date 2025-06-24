# Proposal: Open Community Notes Lexicon

## **1\. Introduction**

This document proposes a lexicon for Open Community Notes, as outlined in the main [Open Community Notes Architecture proposal](https://github.com/johnwarden/open-community-notes/tree/master/proposals/001-architecture). 

The proposed lexicon namespace is **org.opencommunitynotes**.

## **2\. Note Record (org.opencommunitynotes.note)**

This record represents a single community note created by a user to add context to another piece of content on the network.

### **2.1. Spec**

| Field Name | Type | Required | Description |
| :---- | :---- | :---- | :---- |
| subject | com.atproto.repo.strongRef | Yes | A strong reference to the record that the note is about. |
| classification | string | Yes | A mandatory classification for why the post may be misleading, based on the author's selection. |
| text | string | Yes | The user-written explanation providing context. |
| sourceUri | string (format: uri) | Yes | A mandatory URI pointing to a source that substantiates the note. |
| contributorId | string | Yes | The persistent, anonymous identifier for the user who created the note. |
| createdAt | string (format: datetime) | Yes | The timestamp of when the note was created. |

### **2.2. Defined Values for classification**

The classification field must contain one of the following string keys:

| Lexicon Key | UI Text |
| :---- | :---- |
| factual\_error | It contains a factual error |
| altered\_media | It contains a digitally altered photo or video |
| outdated\_information | It contains outdated information that may be misleading |
| misrepresentation\_or\_missing\_context | It is a misrepresentation or missing important context |
| unverified\_claim\_as\_fact | It presents an unverified claim as a fact |
| joke\_or\_satire | It is a joke or satire that might be misinterpreted as a fact |
| other | Other |

### **2.3. Example JSON**

{  
  "$type": "org.opencommunitynotes.note",  
  "subject": {  
    "uri": "at://did:plc:xxxxxxxxxxxx/app.bsky.feed.post/3kabc123xyz",  
    "cid": "bafyreibxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"  
  },  
  "classification": "misrepresentation\_or\_missing\_context",  
  "text": "This quote is accurate, but it omits the following sentence which provides critical context about the speaker's intent.",  
  "sourceUri": "https://example.com/archive/full\_speech\_transcript.html",  
  "contributorId": "anon:ab34fec9de56",  
  "createdAt": "2025-06-25T15:30:00.000Z"  
}

## **3\. Vote Record (org.opencommunitynotes.vote)**

This record represents a user's rating on the helpfulness of a specific community note. Aggregated votes are used to determine which notes are displayed.

### **3.1. Spec**

| Field Name | Type | Required | Description |
| :---- | :---- | :---- | :---- |
| subject | com.atproto.repo.strongRef | Yes | A strong reference to the org.opencommunitynotes.note being voted on. |
| helpfulness | string | Yes | The helpfulness rating. Must be one of "helpful", "somewhat\_helpful", or "not\_helpful". |
| reasons | string\[\] | No | An optional array of predefined reasons justifying the rating. |
| contributorId | string | Yes | The persistent, anonymous identifier for the user casting the vote. |
| createdAt | string (format: datetime) | Yes | The timestamp of when the vote was cast. |

### **3.2. Defined Values for reasons**

The list of valid reasons changes depending on the helpfulness rating.

**Valid reasons if helpfulness is "helpful" or "somewhat\_helpful":**

| Lexicon Key | UI Text |
| :---- | :---- |
| cites\_high\_quality\_sources | Cites high-quality sources |
| easy\_to\_understand | Easy to understand |
| addresses\_claim | Directly addresses the post's claim |
| provides\_important\_context | Provides important context |
| neutral\_or\_unbiased | Neutral or unbiased language |
| other | Other |

**Valid reasons if helpfulness is "not\_helpful":**

| Lexicon Key | UI Text |
| :---- | :---- |
| unreliable\_sources | Sources not included or unreliable |
| sources\_dont\_support\_note | Sources do not support note |
| incorrect\_information | Incorrect information |
| opinion\_or\_speculation | Opinion or speculation |
| typos\_or\_unclear | Typos or unclear language |
| misses\_key\_points | Misses key points or irrelevant |
| argumentative\_or\_biased | Argumentative or biased language |
| note\_not\_needed | Note not needed on this post |
| harassment\_or\_abuse | Harassment or abuse |
| other | Other |

### **3.3. Example JSON**

{  
  "$type": "org.opencommunitynotes.vote",  
  "subject": {  
    "uri": "at://did:plc:yyyyyyyyyy/org.opencommunitynotes.note/3kdef456abc",  
    "cid": "bafyreibbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"  
  },  
  "helpfulness": "helpful",  
  "reasons": \[  
    "cites\_high\_quality\_sources",  
    "provides\_important\_context"  
  \],  
  "contributorId": "anon:de98bca76f54",  
  "createdAt": "2025-06-25T15:35:00.000Z"  
}

## **4\. Design Decisions and Rationale**

* **Lexicon Namespace:** The org.opencommunitynotes namespace was chosen over app.bsky.\* to reflect the project's independent, community-driven nature. This follows standard open-source etiquette, avoids claiming an official namespace without permission, and provides a clean path for future integration should the Bluesky team choose to adopt the feature.  
* **Anonymous-by-Design:** The architecture was changed to store records on a specialized "Community Notes PDS" rather than individual user repositories. This necessitated the contributorId field. This explicit, anonymous identifier allows the system to build a contributor reputation model (crucial for algorithms like bridging) without exposing the public DIDs of note authors and voters, thus protecting them from potential harassment.  
* **Verifiable Links:** The use of com.atproto.repo.strongRef (containing both a URI and a CID) for the subject field is a critical security feature. The URI locates the content, while the CID verifies its integrity. This guarantees that a note is always linked to the *exact version* of the content it was written about, preventing context-drifting and manipulation.  
* **High-Fidelity to UI:** The classification options for notes and the reasons for votes are mapped directly from the provided UI screenshots of X's Community Notes. This ensures the data model is a precise representation of the intended user experience, making front-end development more straightforward.  
* **Structured Data Fields:** Using an array of predefined string keys for reasons (e.g., unreliable\_sources) instead of allowing free text is intentional. This structured approach makes the data easy to aggregate, query, and analyze, which is essential for the algorithms that will determine a note's overall helpfulness score.