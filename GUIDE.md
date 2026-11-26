# Multi-Agent Grid Terminal macOS beta guide

MGT gives you one terminal workspace for coding agents, shells, tests, and logs. A background server keeps your panes running when you detach. The sidebar shows activity for recognized agent CLIs so you can see when a pane is working or needs input.

## Install and start

Check [MGT Releases](https://github.com/hussainiquyyum/mgt-downloads/releases) for a published macOS beta. If GitHub shows an empty Releases page, the installer is still pending.

1. Open the published beta release and download the macOS `.pkg` under **Assets**. The same package supports Apple Silicon and Intel. Check the release notes for supported macOS versions.
2. To check the download, save the published `SHA256SUMS` next to the package and run `shasum -a 256 -c SHA256SUMS` from that folder. Open the package in Finder and follow macOS Installer. It installs the executable in `/usr/local/bin`.
3. Open Terminal. Run `mgt --version` to verify the install. If your shell already has an unrelated `mgt` command, try `mgt-lm --version`. Verify that the output starts with **Multi-Agent Grid Terminal**. You can always run `/usr/local/bin/term-lm --version`.
4. Change into a project directory and run `mgt`. Use the command that reported the MGT version wherever this guide shows `mgt`. Use a small test project for your first session.

## Workspaces and panes

A **workspace** groups panes for a project. A **tab** gives you another pane layout in the same workspace. A **pane** is a terminal that can run an agent CLI or an ordinary command. Click a pane to focus it, right-click for layout actions, and drag the border between panes to resize them. For keyboard shortcuts, press `Ctrl+B`, release it, then press `?`.

Use separate panes for an agent and a test runner, or run two agents side by side. Choose your provider, account, and model in the agent CLI. Install and sign in to each CLI separately. For example, run `claude` in one pane and `codex` in another.

**Working** indicates observed activity, **Blocked** means MGT detected a question or approval, and **Done** or **Idle** indicates a settled state. **Unknown** means MGT could not classify the state. Read the pane to confirm an answer or approval prompt. To detach, close the Terminal window or press `Ctrl+B`, then `q`. Run `mgt` again to reattach.

## Pass work between two agents

For a manual handoff, give Claude Code a bounded prompt in its pane:

> Read this project’s README. Propose a three-step plan to improve its getting-started instructions. Identify any assumptions and leave files untouched.

When it replies, copy the plan into Codex’s pane. Include the goal, scope, and desired response:

> Review the plan below against this project’s README. Keep the review read-only. List up to three missing or inaccurate instructions, and cite the relevant sections. If the plan is sound, say so.

Paste the plan under that prompt. Each CLI keeps its own conversation. The receiving agent gets the text you send and uses the project files and permissions available to its own CLI.

For a CLI handoff, use a shell pane and leave two other panes at their shell prompts. Run `mgt pane list` and use the IDs of those two panes in place of the quoted placeholders below:

```sh
mgt agent start planner --kind claude --pane 'CLAUDE_PANE_ID'
mgt agent start reviewer --kind codex --pane 'CODEX_PANE_ID'
mgt agent prompt planner "Read the README and propose three improvements. Do not edit files." --wait --timeout 120000
mgt agent read planner --source recent-unwrapped --lines 100
```

Copy only the plan from the terminal output. Replace the example text below with it, then submit the review prompt:

```sh
review_prompt=$(cat <<'PROMPT'
Review this plan against the project README. Stay read-only and list up to three gaps.

Paste the plan here.
PROMPT
)
mgt agent prompt reviewer "$review_prompt" --wait --timeout 120000
mgt agent read reviewer --source recent-unwrapped --lines 100
```

If startup pauses for sign-in or an approval, focus that pane and inspect the prompt. Keep approvals with the person using the CLI. If a wait times out, inspect the pane before retrying since the first prompt may have reached the agent. A wait tracks observed activity and may settle at an approval or before an agent's background jobs finish.

## Update and give feedback

MGT shows its version at the bottom of the expanded sidebar and checks for updates when you open it. Select **Update available** to see the signed installer for that version. Open the menu with your prefix key followed by F10 to find the same action with a compact sidebar. Run `mgt update` to check from the command line, or browse the [MGT releases](https://github.com/hussainiquyyum/mgt-downloads/releases).

Detach before opening the installer, then reopen MGT to reconnect. Save in-progress work before restarting a background server if MGT asks. To turn off startup checks, set `version_check = false` under `[update]` in your configuration.

### Upgrading to MGT

If you use TERM-LM 0.1.0, open the signed MGT installer once to switch to the new update source. Your settings and saved workspaces carry over. The installer keeps a `term-lm` compatibility command, and can preserve a verified older installation as `term-lm.pre-mgt`.

If an older command still takes precedence, run `/usr/local/bin/term-lm` and check the migration notice for the path. Use the original package manager to remove a managed launcher, or move an unsigned or custom launcher aside after confirming it belongs to your earlier installation. If the installer reports a conflict at `/usr/local/bin/term-lm`, resolve that path before retrying.

Use [GitHub Issues](https://github.com/hussainiquyyum/mgt-downloads/issues/new/choose) for bugs and feature requests. For bugs, include the MGT version, macOS version and architecture, terminal and agent CLI version, steps to reproduce, and what you expected and observed. Issues are public. Remove credentials, private prompts, repository contents, and sensitive terminal output.
