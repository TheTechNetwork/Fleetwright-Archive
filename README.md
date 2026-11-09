# Fleetwright archive

A starter for the repository Fleetwright pushes your sessions to before their
container or their machine goes. Make yours from it, **make it private**, link
it, and every session you start afterwards is kept here on a branch of its own.

The design and the reasoning are in [`docs/linked-repos.md`](https://github.com/TheTechNetwork/Fleetwright/blob/main/docs/linked-repos.md#the-archive)
in the Fleetwright repository. This file is the setup, and once your copy has
sessions in it, the map.

## Make yours

1. **Use this template**, at the top of this page, then **Create a new
   repository**, and choose **Private**. This is the reason it is a template
   and not something to fork: a fork of a public repository stays public, and
   a copy made from a template is yours, private from its first commit.
2. Install the Fleetwright GitHub App on the repository's owner and select the
   repository, with **Contents: Read & write**.
3. In the Fleetwright app, under **You › Linked repositories › Archive**, enter
   it and press **Check and link**. The check refuses a public repository
   outright, and says whether the App can write there and whether you can push.

Linking changes the sessions started afterwards. The ones already running keep
what they started with.

## What lands here

One branch per session, never shared:

```
fleetwright/<host>/<session>-<started, UTC>
```

Each branch starts from nothing rather than from `main`, so every branch holds
only its session. `main` holds this file and `INDEX.md`: one line per session
that has ended, newest first, linking to its branch. A push is one commit with
the whole of what could be collected at that moment, so **the newest commit on
a branch is the one to read**.

| file | what | when it is missing |
|---|---|---|
| `README.md` | what the session was, and what this commit does and does not hold, in words | never |
| `session.json` | name, title, brief, host, who started it, when, why this push was made, and `workspace`: the commit, branch and remote the patch applies to | never |
| `pane.txt` | the console's scrollback | the session was not running |
| `transcript.jsonl` | the conversation, as Claude Code wrote it | before the session's first turn, or over 6MB |
| `workspace.patch` | what changed in the workspace, as a binary patch | over 6MB of changes: left out whole, never cut |

Pushes happen each time a session comes back to its prompt (at most once a
minute), before it is stopped, forgotten or purged, every ten minutes while it
runs, and when a runner is told to stop. A push that would change nothing makes
no commit. A runner killed hard in the middle of a long turn loses at most the
ten minutes since its last push.

## Getting a session back

```sh
git fetch origin 'refs/heads/fleetwright/*:refs/remotes/origin/fleetwright/*'
git branch -r --list 'origin/fleetwright/*'                 # every session
git show origin/fleetwright/<host>/<session>-<when>:README.md
```

To put a workspace back, apply its patch on the **same commit** the session
was working on. `session.json` names it under `workspace.commit`, with the
branch and the remote, and the branch's README gives the commands.

```sh
s=origin/fleetwright/<host>/<session>-<when>
git show $s:session.json                       # workspace.commit, .remote
git show $s:workspace.patch > /tmp/ws.patch
git -C /path/to/a/clone checkout <workspace.commit>
git -C /path/to/a/clone apply --binary /tmp/ws.patch
```

For a workspace that was not a git checkout, the patch adds every file
against nothing, so the same `git apply` in an empty directory recreates it.

## What to know before relying on it

- **Everything in the workspace is in the patch**, a `.env` included, and
  the whole conversation is in the transcript. That is
  why the archive has to be private, and why Fleetwright asks GitHub whether
  this repository is private at the moment of every push, not only when it
  was linked. Make it public and the pushes stop.
- **Pushes are made as you**, with your own GitHub credential, so they land
  only where you could push yourself.
- **A runner on an owner outside the fleet's minting owners** has no token to
  push with, and says so on the session's page.
- **Nothing runs here.** There is no workflow in this repository on purpose:
  Actions minutes are metered in a private repository, and nothing about an
  archive needs one. Old branches are yours to delete when you are done with
  them.
