# TERM-LM macOS beta guide

TERM-LM gives you one terminal workspace for coding agents, shells, tests, and logs. A background server keeps your panes running when you detach. The sidebar shows activity for recognized agent CLIs so you can see when a pane is working or needs input.

## Install and start

Check [TERM-LM Releases](https://github.com/hussainiquyyum/term-lm-downloads/releases) for a published macOS beta. If GitHub shows an empty Releases page, the installer is still pending.

1. Open the published beta release and download the macOS `.pkg` under **Assets**. The same package supports Apple Silicon and Intel. Check the release notes for supported macOS versions.
2. To check the download, save the published `SHA256SUMS` next to the package and run `shasum -a 256 -c SHA256SUMS` from that folder. Open the package in Finder and follow macOS Installer. It installs `term-lm` in `/usr/local/bin`.
3. Open Terminal. Run `term-lm --version` to verify the install. If your shell cannot find it, try `/usr/local/bin/term-lm --version`, then add `/usr/local/bin` to your shell’s PATH.
4. Change into a project directory and run `term-lm`. Use a small test project for your first session.

## Workspaces and panes

A **workspace** groups panes for a project. A **tab** gives you another pane layout in the same workspace. A **pane** is a terminal that can run an agent CLI or an ordinary command. Click a pane to focus it, right-click for layout actions, and drag the border between panes to resize them. For keyboard shortcuts, press `Ctrl+B`, release it, then press `?`.

Use separate panes for an agent and a test runner, or run two agents side by side. Choose your provider, account, and model in the agent CLI. Install and sign in to each CLI separately. For example, run `claude` in one pane and `codex` in another.

**Working** indicates observed activity, **Blocked** means TERM-LM detected a question or approval, and **Done** or **Idle** indicates a settled state. **Unknown** means TERM-LM could not classify the state. Read the pane to confirm an answer or approval prompt. To detach, close the Terminal window or press `Ctrl+B`, then `q`. Run `term-lm` again to reattach.

## Pass work between two agents

For a manual handoff, give Claude Code a bounded prompt in its pane:

> Read this project’s README. Propose a three-step plan to improve its getting-started instructions. Identify any assumptions and leave files untouched.

When it replies, copy the plan into Codex’s pane. Include the goal, scope, and desired response:

> Review the plan below against this project’s README. Keep the review read-only. List up to three missing or inaccurate instructions, and cite the relevant sections. If the plan is sound, say so.

Paste the plan under that prompt. Each CLI keeps its own conversation. The receiving agent gets the text you send and uses the project files and permissions available to its own CLI.

For a CLI handoff, use a shell pane and leave two other panes at their shell prompts. Run `term-lm pane list` and use the IDs of those two panes in place of the quoted placeholders below:

```sh
term-lm agent start planner --kind claude --pane 'CLAUDE_PANE_ID'
term-lm agent start reviewer --kind codex --pane 'CODEX_PANE_ID'
term-lm agent prompt planner "Read the README and propose three improvements. Do not edit files." --wait --timeout 120000
term-lm agent read planner --source recent-unwrapped --lines 100
```

Copy only the plan from the terminal output. Replace the example text below with it, then submit the review prompt:

```sh
review_prompt=$(cat <<'PROMPT'
Review this plan against the project README. Stay read-only and list up to three gaps.

Paste the plan here.
PROMPT
)
term-lm agent prompt reviewer "$review_prompt" --wait --timeout 120000
term-lm agent read reviewer --source recent-unwrapped --lines 100
```

If startup pauses for sign-in or an approval, focus that pane and inspect the prompt. Keep approvals with the person using the CLI. If a wait times out, inspect the pane before retrying since the first prompt may have reached the agent. A wait tracks observed activity and may settle at an approval or before an agent's background jobs finish.

## Update and give feedback

Detach, download the next signed package from [TERM-LM Releases](https://github.com/hussainiquyyum/term-lm-downloads/releases), and install it over the previous version. Reopen TERM-LM and confirm the version with `term-lm --version`. Save in-progress work before restarting a background server if TERM-LM asks you to do so.

Use [GitHub Issues](https://github.com/hussainiquyyum/term-lm-downloads/issues/new/choose) for bugs and feature requests. For bugs, include the TERM-LM version, macOS version and architecture, terminal and agent CLI version, steps to reproduce, and what you expected and observed. Issues are public. Remove credentials, private prompts, repository contents, and sensitive terminal output.
