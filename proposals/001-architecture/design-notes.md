# Single Community Notes Service

The community notes service needs to store proposed notes and ratings in an internal database. It will then publish proposed notes and ratings records to atproto: but there are some special requirements for how these records are published.

- Notes and ratings are anonymous. They are all signed using a single service account DID, with an additional anonymous IDs fields.
- Only users with the required rating impact score can propose notes (submit note records).
- Ratings are only published to atproto *after* the note has achieved a status of helpful or not helpful.

## Reading Note and Rating Records

The Community Notes service will also provide an API that returns notes needing ratings (e.g. given a post ID). And this API needs to return not just the Note, but the NoteView, which is a Note "hydrated" with additional information about the viewer (user) -- specifically, any existing rating for that user on that note. Hydration would typically function of an AppView in atproto -- this is how posts are hydrated along with information about whether the user has liked the post -- but since ratings aren't immediately published to atproto they won't be available to regular app views. Instead the API for getting NoteViews must be an authenticated request to the Community Notes app.

Finally, the service will publish useful notes (labels with text), as well as metadata about notes (needs more ratings, etc.). 
 
This all point to a single Community Notes service that has features of a PDS, App View, and Labeler. 
