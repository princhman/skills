---
name: agentline-linear
description: Use for anything that reads or writes Linear - reading issues, commenting, asking the user a question, posting proof, changing status or labels, filing bugs, or waiting for the user's reply on an issue. Linear goes through the `agentline` MCP tools so posts appear as Agentline, and issue_watch replaces polling.
---

# Linear through Agentline

The `agentline` MCP server is the only Linear connection. Everything it posts
shows as **Agentline**; the user's own comments stay theirs. That is how the
user, and the watcher, tell agent messages from human ones.

- Never post to Linear any other way (Linear's own MCP, the API with a personal
  key, `gh` comments mirrored to Linear). Those post as the user.
- If the `agentline` MCP tools aren't loaded, say so in your report. Don't fall
  back to another Linear tool for writes, and don't schedule polling.

## Tools

| Need | Tool |
| --- | --- |
| Read an issue + its comments | `issue_get` (comments have `byAgentline` true/false, oldest first) |
| Find issues | `issue_list` (team, project, states, labels, query, updatedSince, limit) |
| See a screenshot on an issue | `file_fetch` with its `uploads.linear.app` URL |
| File an issue | `issue_create` |
| Status, labels, title, description, priority | `issue_update` |
| Say something to the user | `comment_create` (`parentId` to reply in a thread) |
| Fix your own comment | `comment_update` |
| Attach proof | `file_upload`, then a comment |
| Wait for the user's reply | `issue_watch`; stop with `issue_unwatch` |
| A reply didn't arrive | `issue_watch_status` (each delivery's state and last error) |

States, labels, projects and teams are names, matched case-insensitively. A
wrong name returns the valid ones; pick from that list, never invent one.

## Asking and waiting

1. `comment_create` with one specific question and the options you see, each
   with what you'd do.
2. `issue_update` with `addLabels: ["Needs input"]` (or the repo guide's
   waiting state).
3. `issue_watch` with the issue, the thread that should act on the answer, and
   the machine that thread runs on (`mac` or `minipc`). Your own thread ID is
   `currentThreadId` from the T3 thread tools. A delegate that ends after its
   report doesn't watch; it names the waiting issue in the report and the
   orchestrator watches with its own thread.
4. Move on. Don't schedule checks or sleep; the reply arrives as a new message.

Same for visual review: after moving an issue to In Review, watch it.

## When a reply arrives

It comes in as a message starting `New Linear comment on PRI-…`, with the
comment quoted and its ID. Then:

1. `issue_get` the issue. Read the whole thread, not just the quote; the user
   may have replied twice or attached a screenshot (`file_fetch` it).
2. Remove `Needs input` with `issue_update` when you pick the issue up again.
3. Act, then answer with `comment_create`. To stay in the user's thread, pass
   `parentId`: the comment's own parent if it has one, otherwise its ID (Linear
   threads are one level deep).
4. `issue_unwatch` once nothing on the issue waits for the user any more
   (answered and building, merged, or closed). Watches expire after 14 days.

If a reply you expected hasn't arrived, check `issue_watch_status` before
anything else and report its error. Calling `issue_watch` again retries failed
deliveries. Keep any fallback check until one reply has actually arrived.

Only new comments by people are delivered (replies inside a comment thread count). Edited comments, status changes and
Agentline's own posts are not.

## Proof

```bash
f=proof.mp4
# file_upload {filename: "proof.mp4", contentType: "video/mp4", size: <bytes>}
#   -> uploadUrl, assetUrl, headers
curl -sf -X PUT "$uploadUrl" -H "Content-Type: video/mp4" -H "Cache-Control: public, max-age=31536000" \
  <one -H per returned header> --data-binary @"$f"
```

PUT within 60 seconds of `file_upload`, with every returned header. Then
`comment_create` saying what each file shows, with `![before](assetUrl)` or a
plain link for videos. Delete the local file afterwards.

## Writing comments

The user reads these on a phone. Get to the point.

- Lead with the answer or the result in one sentence. No greeting, recap of
  the request or narration of what you did step by step.
- At most ~5 short lines or bullets. A question asks one thing and lists the
  options in a line each.
- Proof: one line per file saying what it shows. Nothing else.
- Leave out SHAs, branch names, file paths, test counts and tool output unless
  the user asked. Those go in the PR or your report, not Linear.
- Plain words, no headings, no tables unless comparing options side by side.

## Don't

- Don't copy Linear's `gitBranchName` prefix (`novvladkyiv/…`) into branch
  names; follow the repo guide.
- Don't write `Fixes PRI-n` for an issue you didn't fix or reproduce; use
  `Refs PRI-n` so merging doesn't close it.
- Don't post progress chatter. One comment per question, per proof set, per result.
