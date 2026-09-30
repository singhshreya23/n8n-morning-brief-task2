# Task 2: Morning Brief Workflow (n8n)
**Author:** Shreya Singh · **File:** `Task2_Workflow_ShreyaSingh.json`

## What it does
Every hour, the workflow fetches today's front-page stories from Hacker News (a popular tech news site). It keeps the 5 with the most points, looks up each story's author, and posts one summary message to a Discord channel.

## APIs used and why
| # | API | What it returns | Why I chose it |
|---|---|---|---|
| 1 | Hacker News Search API: `hn.algolia.com/api/v1/search?tags=front_page` | Front-page stories with title, link, points, comments, author | Free, no sign-up or API key, simple JSON |
| 2 | Hacker News Firebase API: `hacker-news.firebaseio.com/v0/user/<author>.json` | The author's profile, including karma (reputation score) | Adds information the first API doesn't give (enrichment) |
| 3 | Discord Webhook | Posts the message into a channel | Free, instant, and easy for a team to read |

## Transformation (built-in n8n nodes plus expressions, no custom code)
1. **Split Into Stories** turns the single API response into one item per story.
2. **Sort by Points** puts the highest-scoring stories first.
3. **Keep Top 5** drops everything after the 5th story.
4. **Keep Useful Fields** keeps only title, link, points, comments and author, and falls back to the Hacker News discussion link if a story has no URL.
5. **Get Author Karma**, the second API call, runs once for each of the 5 stories. **Add Karma to Story** joins the author's karma onto the story.
6. **IF Points > 100:** stories above 100 points are formatted for the brief. Stories at or below 100 go to a "Skip" branch and are left out.
7. **Combine Into One List** joins the lines, and the Discord node sends one message.

## Error handling (never fails silently)
- **Automatic retries:** the story fetch retries 3 times, and the karma and Discord calls retry twice.
- **Story fetch fails, or Discord send fails:** the node's red **On Error** output sends a alert to Discord with the error message and time. The **Mark Run as Failed** node then makes the run show as *Failed* (red) in n8n's execution list, so it is always visible.
- **Karma lookup fails:** the story is still sent, with karma shown as `unknown`. One small failure shouldn't cancel the whole brief.
- **How I tested it:** I temporarily broke the URL in *Get Front Page Stories*. The Discord alert arrived and the run was marked failed.

## Credentials
No passwords or links are typed into the nodes. The Discord webhook URL is stored in n8n's **Credentials** store as a *Discord Webhook* credential, and both Discord nodes reference it. The Hacker News APIs are public and need no key.
