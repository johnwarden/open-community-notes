# The Open Community Notes Standard

This repository contains proposals and specs for an open Community Notes standard based on AT Protocol. 

## How to Contribute

Your involvement is highly encouraged! Feel free to:

- Open an issue to discuss suggestions, concerns, or ideas.
- Submit pull requests with improvements, edits, or clarifications.
- Share feedback or engage in ongoing discussions via issues or discussions.

## License

All content in this repository is licensed under the [MIT License](LICENSE), encouraging widespread collaboration and adoption.

# Overview 

The vision is a version of X's Community Notes that:

1. Is [**open with credible exit**](https://perma.cc/LC9R-Q6JY). Although Community Notes requires some [*architectural centralization*](https://medium.com/@VitalikButerin/the-meaning-of-decentralization-a0c92b76a274)—some entity must run the Community Notes algorithm on the full dataset—all notes and ratings can live in an open network. This enables anyone to independently verify the algorithm’s output or run their own service.

2. Could be used by any social network—from federated networks like AT Protocol and ActivityPub, to TikTok and even X itself.

3. Can be used as a general protocol for community moderation.

We propose closely mirroring X's Community Notes implementation, using their open-source algorithm and a similar interface design. We'll also adopt their solutions for handling **anonymity, bots, and manipulation**, though implementing these features in an open protocol presents unique challenges. See the [Challenges](#challenges) section below for our proposed solutions.

### Based on AT Protocol

We propose building the Open Community Notes protocol on top of [AT Protocol](https://en.wikipedia.org/wiki/AT_Protocol). AT Protocol supports different [**social modes**](https://bsky.social/about/bluesky-and-the-at-protocol-usable-decentralized-social-media-martin-kleppmann.pdf), beyond just the microblogging mode used by the Bluesky app. Using a [Custom Schema](https://docs.bsky.app/docs/advanced-guides/custom-schemas), Open Community Notes can use AT Protocol as an open **store for notes and ratings**. Community Notes **apps** would allow users to browse, propose, and rate notes. And an independent Community Notes **service** would be implemented as an [App View](https://docs.bsky.app/docs/advanced-guides/federation-architecture#app-views), allowing Bluesky and others to easily display helpful Community Notes in their apps.

### Multi-Platform

This Open Community Notes wouldn't be limited to Bluesky and other AT Protocol apps. While AT Protocol would serve as the data layer, a Community Notes App would let users submit, browse, and rate notes on **any content identifiable by a URI** (such as a Tweet). The feed of helpful Community Notes would be available as a **web service** that anyone can subscribe to.

### Generalized Moderation

A generalized community moderation protocol could use the same algorithm that Community Notes uses for identifying helpful *notes* for identifying helpful **labels with notes**. A classic Community Note would be a `annotation` label along with a note. But the algorithm could also be used for `harassment` or other labels, with optional notes explaining the reason for the label.

The lexicon for [PMsky](https://pmsky.social/) was built on this idea of generalized community moderation. Open Community Notes uses the [social.pmsky lexicon](https://docs.pmsky.social/tech/lexicon).

## High-Level Architecture

![open-community-notes.png](open-community-notes.png)

### **Community Notes App(s)**

A user interface for proposing notes, browsing notes that need ratings, and voting on notes. Interfaces with the Community Notes Service to submit notes and ratings, and with the Scoring Service to pull helpful notes and notes needing ratings.

### **Community Notes Service**

Provides API endpoints for proposing and voting on notes. Publishes signed note and ratings records using an **Anonymous IDs** kept separate from users' public profiles. See the [Anonymous IDs proposal](/003-aids#readme).

The service would also implement the ATProto Labeler API, publishing community notes labels..

### **Community Notes Aggregator**

Runs the Community Notes algorithm to score notes and produce rater impact scores.

### Integrated Social Apps

Apps like Bluesky can use display notes under posts that have a community notes label. Since the labeler is a web services, even non-AT Protocol apps like Mastodon can integrate them.

Apps can either build Community Notes features directly into their interface or direct users to an external app for writing and rating notes.

## Generalized Bridging-Based Moderation

Community Notes allows members of the community to add context to posts. However, the same [bridging-based ranking algorithm](https://jonathanwarden.com/understanding-community-notes/) used to identify helpful notes could be used to identify [*any*](https://github.com/bluesky-social/social-app/issues/5783#issuecomment-2495557772) moderation label (scam, harassment, etc.). Further, any label could be associated with additional [context](https://github.com/bluesky-social/social-app/issues/4003) (a reason for that label).

So, a generalized Community Notes protocol would be used to add labels to posts, along with notes/reasons for those labels.

### Labels with Notes

A Community Note can just be thought of as an ATProto `annotation` label with an additional `note` field with the text of the note. Community Notes-enabled apps can display these labels as community notes when it sees them.

X's Community Notes also places a "Rate Proposed Community Notes" notice below posts that have notes that have not yet been rated as helpful/unhelpful. Community-notes enabled apps can also display these prompts for posts with a `proposed-annotation` label.

See the proposed [Labeling Architecture](/004-labeling#readme) for more details.

### Adding Reasons to Labels

A moderation service that publishes labels such as `scam` might sometimes want to include an explanation for why the post is a scam. So in general, a note can be considered as a *reason for a label.* So moderation services, including the Community Notes Scoring Service, could optionally publish notes with any label.

## Challenges

### Anonymity

X's Community Notes preserves anonymity of contributors. Each contributor receives an ID, and while their notes and ratings are public, X doesn't publish information linking these IDs to X handles. We propose that the Community Notes Service manage users’ anonymous IDs and keep the link between anonymous IDs and users' public accounts secret. There are various ways this can be done. See [Anonymous IDs below](#anonymous-ids).

### Manipulation

The Community Notes algorithm has some resistance to Sybil attacks and coordinated manipulation. However, it isn't manipulation-proof—too many bot accounts will break it.

X uses various methods of defending against manipulation.

**Proof of Personhood**

* X requires telephone numbers for signups, imposing a small cost that discourages bots. Community Notes Apps could require this, but it can't be enforced by the protocol.

* **Possible Alternative Solution: Proof of Work:** An alternative would be requiring a small *proof of work* from new contributors (see [Anubis](https://anubis.techaro.lol)). Regular users would complete this computation on their device at negligible cost, while Sybil attacks could become prohibitively expensive.

**Delayed Publishing**

* To prevent users from simply copying ratings from other users, X delays publishing user ratings. Community Notes PDSs could also impose a delay between the time they receive a rating through the note rating API endpoint and the time they write a record to the PDS.

**Trust and Rating Impact**

* Community Notes users on X earn a Rating Impact score based on how often their ratings help identify helpful or unhelpful notes. New contributors must achieve a minimum rating impact before they can write notes.

* This same requirement could be enforced independently by the scoring service, PDS, and UIs.

**Core Contributors**

* Community Notes at X began with a core group of committed early adopters who acted in good faith. By the time the system faced widespread attention and potential manipulation, the community had established clear unwritten norms about what makes a note helpful. The rating impact system required new contributors to adapt to these norms.

* Organizations building the first Open Community Notes apps could similarly implement a controlled rollout, seeking to attract a diverse foundation of trustworthy early users.

