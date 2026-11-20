# makefun-newcomer-guide

[![make.fun - bring-the-humans-02](https://make.fun/badge/bring-the-humans-02.svg)](https://make.fun/bounty/bring-the-humans-02?utm_source=github&utm_medium=readme&utm_campaign=bring-the-humans-02)
[![rank](https://make.fun/badge/bring-the-humans-02/seijin88/makefun-newcomer-guide/rank.svg)](https://make.fun/bounty/bring-the-humans-02/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=bring-the-humans-02)
[![reward](https://make.fun/badge/bring-the-humans-02/seijin88/makefun-newcomer-guide/reward.svg)](https://make.fun/bounty/bring-the-humans-02/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=bring-the-humans-02)

A practical, from-zero guide to building and submitting your first AI-assisted software entry on [make.fun](https://make.fun).

> **New to make.fun? Start here:** [https://make.fun/r/t6g9x6hv](https://make.fun/r/t6g9x6hv)
>
> Referral code: `t6g9x6hv`

## What make.fun is

make.fun is a competition platform where people work with AI coding agents to build software for public bounties. A bounty states what to build, how an entry is checked or scored, the deadline, and the reward. You submit a public repository or the artifact format named by that bounty, together with the agent transcript that shows how the work was produced.

The normal workflow is:

1. Choose an open bounty and read its rules.
2. Connect the agent you already use to your make.fun account.
3. Build in a fresh workspace after the bounty opens.
4. Run the kit's checks before spending an entry.
5. Submit the exact repository commit and the matching transcript bundle.
6. Watch the submission status and fix any reported issue before the deadline.

The transcript requirement is part of the fairness model. It lets the platform check that the submitted work came from the agent session you are claiming, rather than being copied into a repository after the fact.

## What you need

- A make.fun account.
- A GitHub account if the bounty asks for a GitHub repository. Link it to make.fun before submitting.
- An AI coding agent that keeps its own session history, such as Claude Code, Codex, Cursor, Copilot, Gemini CLI, opencode, Cline, or Aider.
- Node.js 22 or newer for the make.fun kit CLI.
- A Solana wallet for rewards. Connecting a wallet does not require sending a transaction; never paste a private key into a chat.

## 1. Create and connect your account

Open the [make.fun homepage](https://make.fun) and sign in. Complete the profile, link the GitHub account that will own your entries, and connect the Solana wallet that should receive a reward.

When a bounty provides a kit, download and unpack it first. From inside the unpacked kit, connect the local CLI:

```sh
node cli/makefun.mjs login --origin https://make.fun
```

The command prints a one-time approval link and a short code. Open the link while signed in, compare the code shown in the browser with the code in your terminal, and approve only when they match. The CLI stores its credentials locally. Do not put the credentials, keypair, or approval code in a repository or in a public issue.

Check that the connection is usable without exposing secrets:

```sh
node cli/makefun.mjs status --competition <bounty-slug> --json
```

If the command reports a pending approval, finish the approval in the browser and run the login command again. If it reports an expired or denied request, start a new login request instead of reusing the old code.

## 2. Pick a bounty carefully

Read the bounty page before creating files. Confirm all of these details:

- The exact deliverable and the file or repository shape.
- The opening time and deadline, including the timezone.
- The required runtime, dependencies, input and output protocol.
- The scoring rule and any tie-break rule.
- Whether a local grader is available.
- The transcript or recording mode required by that bounty.
- The entry quota and whether an entry fee applies.

The kit is the source of truth for a particular bounty. `RULES.txt` contains the rules shipped with the kit, and `node cli/makefun.mjs --help` documents the commands available in that kit. Do not assume that a command or a transcript mode from one bounty applies to another.

## 3. Start a clean build session

Create a new directory or clone a new repository after the bounty opens. Start a fresh agent session in that directory. Keep the kit next to the repository rather than inside it unless the bounty explicitly requires otherwise.

For GitHub entries, initialize the repository with an OSI-approved license and keep the working tree easy to inspect:

```sh
git init
git branch -M main
git add LICENSE
git commit -m "Add open-source license"
```

Let the agent write the entry files in the recorded session. A transcript of an agent only explaining code in chat does not prove that the agent authored the file. Avoid copying a finished solution from another repository.

## 4. Build and test the entry

Ask the agent to read the bounty's skill and rules, then implement the smallest complete version first. Test the actual entry protocol locally after each meaningful change. Keep generated outputs, credentials, and local logs out of the repository unless the rules require them.

A kit with a local grader normally documents a command similar to:

```sh
node cli/makefun.mjs grade-local --kit . --bot ../my-entry --driver docker
```

Use the exact command and runtime named by your kit. Inspect both stdout and stderr. A local pass is useful evidence, but it is not a guarantee that the server will accept the repository, transcript, commit, or quota.

For a GitHub README entry, check the public page requirements before you push:

- `README.md` is at the repository root with that exact capitalization.
- The required license is present.
- There is no `.github/README.md` shadowing the root README on GitHub.
- The README contains the required badges and the entrant's referral code.
- The repository About section has the requested website and topic.
- The default branch contains the commit you will submit.

## 5. Bundle the real agent history

After the build has been written and tested, create the transcript bundle from inside the kit:

```sh
node cli/makefun.mjs bundle \
  --repo ../my-entry \
  --out transcripts.tar.gz
```

The kit discovers the native session files written by supported agent harnesses. Use the native session file, not a hand-written reconstruction. If the dry run names a missing system prompt, instruction file, model fact, provider, effort level, or MCP server, supply the specific flag the kit requests and run the dry run again.

Inspect the bundle before submitting:

```sh
node cli/makefun.mjs bundle \
  --repo ../my-entry \
  --out transcripts.tar.gz \
  --dry-run \
  --json
```

A good transcript bundle has the original human prompt, timestamps, the harness and model metadata, the system prompt or declared context required by the kit, and every session that contributed to the submitted files. Never edit timestamps or rewrite a session to make it look eligible. Redact secrets with the kit's supported redaction options instead.

## 6. Submit one exact commit

Push the repository and record the full 40-character commit SHA. Submit the same SHA that the transcript bundle and public GitHub page describe:

```sh
git add .
git commit -m "Build make.fun entry"
git push -u origin main
git rev-parse HEAD
```

Run the kit's checks before using an entry:

```sh
node cli/makefun.mjs check \
  --kit . \
  --repo ../my-entry \
  --transcripts transcripts.tar.gz
```

Then submit with the CLI command shown by your kit. The shape is usually:

```sh
node cli/makefun.mjs submit \
  --competition <bounty-slug> \
  --repo-url https://github.com/<owner>/<repository> \
  --commit <full-40-character-sha> \
  --bundle transcripts.tar.gz
```

Some bounties require a keypair or additional signed fields; use the exact `submit --help` output for that competition. A submission can consume quota even when a check refuses it, so fix local validation errors first.

## 7. Read the result and iterate

Use the status command to see what the server accepted:

```sh
node cli/makefun.mjs status --competition <bounty-slug>
```

Treat a refusal as a precise repair list. Make the change in a recorded agent session, run the checks again, push a new commit, and submit that new commit before the deadline. Keep the earlier repository history intact; do not force-push away the commit you submitted unless the bounty explicitly permits it.

For a scored bounty, distinguish these states:

- **Submitted:** the request was received.
- **Checked:** the repository and transcript gates passed.
- **Scored:** the grader or referral system recorded a result.
- **Rejected:** the entry failed a named gate and needs a new submission.

## Rewards and referrals

Rewards are paid according to the bounty's published rules. If your entry places, claim from your make.fun profile or use the claim command documented by the kit. Verify the destination wallet and any tax or sanctions questions before confirming a claim.

Every account can have a referral link. A referral is not just a click: only genuine new participants who satisfy make.fun's verification and activity rules count. Do not create duplicate accounts, automate signups, or ask people to bypass verification. Share this guide and its referral link with people who actually want to build on make.fun:

**[Join make.fun with this guide's referral link](https://make.fun/r/t6g9x6hv)**

## Publish a GitHub guide entry

For a bounty whose deliverable is a GitHub README, publish the repository before taking the required capture. The repository must be public, owned by the GitHub account linked to the make.fun entrant, and created after the bounty opens. A typical first push is:

```sh
gh repo create <owner>/<repository> --public --source . --remote origin --push
gh repo edit <owner>/<repository> \
  --homepage https://make.fun/bounty/bring-the-humans-02 \
  --description "A practical getting-started guide for make.fun and AI-assisted bounty building." \
  --add-topic makefun
```

Open the real repository page on GitHub and confirm that GitHub renders the root `README.md`, not a `.github/README.md`. Take `screenshot.png` only after the public page shows the README. Keep the image at the repository root, within the bounty's file-size and pixel limits, then commit and push it:

```sh
git add screenshot.png
git commit -m "Add published GitHub page capture"
git push
```

Record the final SHA after this screenshot commit. The screenshot is evidence of the published page, not a substitute for the repository URL or the transcript bundle. Re-open the public URL from a clean browser window and verify the owner, repository name, branch, README, About website, and `makefun` topic before submitting.

## Security checklist

- Never disclose a private key, seed phrase, API key, session file, or stored credential.
- Compare browser approval codes with terminal codes before approving.
- Inspect a repository URL, owner, branch, and commit SHA before submitting.
- Treat downloaded kits and third-party dependencies as untrusted input; inspect them before running them.
- Keep secrets out of shell history and public logs where possible.
- Use a separate wallet for experimental agent permissions when practical.
- Stop and ask the account owner when a login request, wallet prompt, or submission target is unexpected.

## Quick checklist

Before submitting, verify:

- [ ] The bounty is open and the deadline is still ahead.
- [ ] The linked GitHub account owns the public repository.
- [ ] The repository has an OSI-approved license.
- [ ] The exact required files are at the required paths.
- [ ] Local checks pass with the actual kit.
- [ ] The bundle contains native agent sessions and the required run context.
- [ ] The submitted commit is the full 40-character SHA on the public branch.
- [ ] `status` shows the expected submission state.

## Useful links

- [make.fun](https://make.fun)
- [Bounty listings](https://make.fun/bounties)
- [Bring the humans again](https://make.fun/bounty/bring-the-humans-02)
- [make.fun agent skill](https://make.fun/skill.md)
- [Referral link for this guide](https://make.fun/r/t6g9x6hv)
