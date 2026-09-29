# Better Texting for Hermes Agent

A small, public [Hermes Agent](https://hermes-agent.nousresearch.com/) skill for conversations that feel like texting: a prompt acknowledgment, useful updates during longer work, and a concise completion message. It works with any messaging channel that can deliver mid-turn assistant messages. Photon/Spectrum iMessage setup is below.

The skill contains instructions only. It has no scripts, dependencies, network calls, scheduled jobs, or access to contacts. It does not send messages on its own.

## What it looks like

**You:** “Can you fix the broken build?”

**Hermes, before using tools:** “Yep — I’ll run the build and see where it fails.”

**Hermes, if there is a meaningful finding:** “Found it. An import points to the old module name.”

**Hermes, after verifying:** “Fixed. The build passes now.”

These are separate assistant messages sent at different points in one task, not one response split by blank lines.

## Install

Install from GitHub using Hermes's skills command:

```text
hermes skills install https://raw.githubusercontent.com/aidenbuildsthings/hermes-better-texting/main/SKILL.md
```

Then invoke it in a messaging chat with `/better-texting`, or ask Hermes to use the better-texting skill. Hermes loads skills on demand, so installation alone does **not** make it run in every conversation.

For consistent behavior, auto-load the skill at the start of every new Hermes session:

```yaml
skills:
  auto_load:
    - better-texting
```

Merge this with any existing `skills.auto_load` list. Auto-load applies across CLI, TUI, gateway, cron, and API sessions; the skill's instructions themselves apply only when Hermes is messaging. This adds the skill text to every new session's context. Config changes take effect when a new session begins.

If you prefer a shorter always-on instruction instead of auto-loading the full skill, append the snippet in [`ALWAYS_ON.md`](ALWAYS_ON.md) to your own `SOUL.md` (normally `~/.hermes/SOUL.md`). Do not replace the rest of your `SOUL.md`. Use **either** approach to start; using both repeats the same guidance.

## Photon / Spectrum: enable separate mid-task bubbles

In `~/.hermes/config.yaml`, set a Photon display override:

```yaml
display:
  platforms:
    photon:
      interim_assistant_messages: true
      tool_progress: off
```

Merge these keys with your existing `display` section. `interim_assistant_messages` lets completed assistant updates appear as separate chat messages. `tool_progress: off` avoids permanent tool-status bubbles such as “Reading file…”. Photon does not support editing sent messages. Restart the Hermes gateway after changing its configuration.

If you already set `display.interim_assistant_messages: true` globally, that also applies to Photon unless a Photon-specific override disables it. The per-platform form keeps the change scoped to Photon.

**Important:** This setting makes *mid-turn* assistant messages deliverable. It does not automatically split a completed final answer into several bubbles. The skill asks Hermes to send distinct thoughts as the task progresses, but exact bubble count depends on the model, Hermes version, and channel. A channel or model that only emits a final response will still produce one bubble.

## What changes

| Component | Effect |
| --- | --- |
| Installing `SKILL.md` | Adds an optional writing and workflow instruction that Hermes can load on demand. |
| Adding `skills.auto_load` or the `ALWAYS_ON.md` snippet | Applies the core rhythm to new sessions using that Hermes profile. These are manual alternatives. |
| Photon config override | Permits mid-turn assistant messages to be delivered separately and hides routine tool-status chatter. This is a manual step. |
| Tasks, tools, contacts, permissions, and credentials | No changes. The skill has no executable code or configuration side effects. |

The intended behavior is an acknowledgment **before** tool work, then a useful update only when something changes or the work takes a while, then a verified result. Simple questions usually stay one bubble. User instructions for silence, detail, or tone take priority.

## Try it

1. Restart the gateway if you changed `config.yaml` or `SOUL.md`, then start a new session so the skill or context change is loaded.
2. In your Photon conversation, ask Hermes to use `/better-texting` and give it a task that actually needs tools.
3. Check that the first bubble arrives before the task finishes, the work continues, and a separate result bubble arrives afterward.
4. Try a simple question and a longer task. The simple question should stay concise; the longer task should get updates only when useful.

If Hermes waits until the end, check that the skill was loaded and that `display.platforms.photon.interim_assistant_messages` resolves to `true`. If it still never produces mid-turn assistant messages, the model may not follow the instruction reliably; a deterministic instant acknowledgment requires a gateway feature, not just a skill.

## Design limits

- **Separate bubbles are transport behavior.** Paragraphs inside a single model response do not become separate messages by virtue of this skill.
- **No guaranteed instant response.** Hermes must first process the request and generate an acknowledgment. The skill improves the pattern but cannot promise a fixed latency.
- **No forced final-answer split.** Hermes's normal final reply is one message. A reliable multi-bubble final answer would require channel or gateway support for an explicit message sequence.
- **No extra sending privileges.** The skill does not use `hermes send` or bypass the gateway's normal delivery and permission rules.

## Sources

- [Hermes skill format and publishing](https://hermes-agent.nousresearch.com/docs/developer-guide/creating-skills)
- [Hermes display settings and mid-turn messages](https://hermes-agent.nousresearch.com/docs/user-guide/configuration)
- [Hermes Photon iMessage documentation](https://hermes-agent.nousresearch.com/docs/user-guide/messaging/photon)
- [Hermes global context files](https://hermes-agent.nousresearch.com/docs/user-guide/features/context-files)
- [Hermes skill auto-load configuration](https://hermes-agent.nousresearch.com/docs/user-guide/cli#persistent-auto-load-via-config)

## License

MIT. See [LICENSE](LICENSE).
