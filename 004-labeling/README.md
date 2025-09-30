# Open Community Notes Labeling Architecture

The Community Notes app will piggyback on Bluesky labelers for displaying helpful proposals, as well as the "rate proposed community notes" prompts.

## Community-Notes-Enabled Social Apps

Community-Notes-Enabled Social Apps will:

- Display helpful community notes below posts
- Show "rate proposed community notes" prompts below proposals that need ratings
- Have a "Write a Community Note" menu option

These apps should be able to use existing Bluesky App Views and PDSs. To find posts with helpful notes or notes needing ratings, the app view will rely on labels.

### App Labeler Approach

The Community Notes labeler will be configured as an **App Labeler** rather than requiring individual user subscriptions:

- Apps will send the Community Notes labeler DID in the `atproto-accept-labelers` header in requests to Bluesky App Views
- This ensures all users automatically receive Community Notes labels without any subscription process
- Labels from app labelers are automatically sent in PostView objects returned by the bsky service (the app view)
- For posts with a 'proposed-annotation' label, the front-end will display a "rate proposed community notes" prompt
- For posts with a 'annotation' label, the front-end will lookup the proposals by calling getProposals in the Community Notes service

This approach:
- ✅ Requires no user action or subscription management
- ✅ Provides consistent experience across all users
- ✅ Follows the same pattern as Bluesky's core moderation services
- ✅ Eliminates complex subscription logic and error handling


## Community Notes Service: getProposals endpoint

The responses from the Bluesky App Views will include posts with labels, but these labels will not include the actual text of the notes. To get the text of the notes, Apps will make an additional call to the Notes service `getProposals` endpoint.

## Community Notes Labeler

- The community notes labeler service will implement the Atproto labeler interface and publish `annotation` and `proposed-annotation` labels
- These labels will not contain the text of the proposal (Bsky app views will ignore these). The `getProposals` endpoint will provide this instead.
- The labeler will publish `neg` labels (e.g. remove labels) when a note that was previously rated helpful changes it status
