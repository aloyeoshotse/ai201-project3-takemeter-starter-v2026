# TakeMeter collection log

Collection contains 230 unique posts in one unsplit file; at least 200 required.

Public Reddit posts were copied through the signed-in Chrome browser from the
new-post listings of r/pcgaming, r/Games, r/gamedev, r/patientgamers,
r/IndieGaming, and r/gamingsuggestions on October 18, 2026.
No Reddit API credentials or blocked anonymous endpoints were used.
Collection started with the compact old Reddit interface. When Chrome blocked
that host with ERR_BLOCKED_BY_CLIENT, collection paused. It resumed through the
normal www.reddit.com interface, opening individual posts to obtain full bodies.
Chrome later blocked another post, at which point further browsing stopped.

Text posts contain the full rendered title, a blank line, and full expanded body.
Link posts contain the full Reddit title only, not the linked article.
Rendered text is preserved without rewriting, shortening, or correcting it;
Reddit formatting is represented as plain text rather than original Markdown.
Image/video/gallery posts, crossposts, recurring moderator threads, duplicates,
deleted posts, and text posts whose full body did not load were excluded.
Collection from new listings is a convenience sample, not a random sample of
Reddit. Some r/Games posts came from its developer showcase on this date.
r/gamingsuggestions was added to broaden the pool of community requests.
The source of a post is not its assigned label: no labels have been inferred.

labels.csv is one unsplit file with text,label,note. Every label and note is blank.
No AI labeling was performed, and no rows have been marked cold.
The student must label 20 posts cold before asking AI to pre-label the rest.
The remaining rows must each be reviewed by the student after any AI pre-labeling.
Source row numbers in collection_sources.csv are 1-based data rows, excluding
the CSV header. Collection has not established the four-label balance.
