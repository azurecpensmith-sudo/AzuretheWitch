# Character Continuity v5.0.1 Open Beta

## Staggered Focus and Persistence — Creator Guide

Character Continuity helps an AI portray recurring NPCs consistently while allowing lasting decisions, relationships, memories, and development to survive across a long adventure.

v5.0.1 removes stored Temporary State. Current feelings, thoughts, goals, and tensions are inferred naturally from the Core cards, current input, Recent Story, AI Instructions, Author's Note, and durable continuity already in context. The script no longer asks the model to maintain a separate short-lived State record.

This is an open beta. Test it first in a copy of an adventure and back up important cards before updating.

> **Compatibility:** Character Continuity requires the Context modifier. Cached models and optimized context are unsupported.

## Quick setup

1. Add the Character Continuity v5.0.1 script as a Library.
2. Connect it once to each Input, Context, and Output modifier.
3. Create one `CC CORE: Full Name` Story Card for every managed NPC.
4. Optionally create `CC PLAYER`, `CC VOICE`, `CC MOTIVES`, and `CC BOUNDARIES` cards.
5. Start the adventure. The script creates its configuration, status, continuity, relationship, memory, and Turning Point cards.
6. Adjust significance and configuration only if the defaults do not fit your cast.

Do not paste the complete script into all three modifiers. If a scenario package already connects it, do not add a second set of calls.

### Modifier calls

Input modifier:

```js
const modifier = (text) => ({
  text: CharacterContinuity("input", text)
});
modifier(text);
```

Context modifier:

```js
const modifier = (text) => ({
  text: CharacterContinuity("context", text)
});
modifier(text);
```

Output modifier:

```js
const modifier = (text) => ({
  text: CharacterContinuity("output", text)
});
modifier(text);
```

## Registering an NPC

Create a Story Card titled exactly:

```text
CC CORE: Full Name
```

Recommended format:

```text
Aliases: Nickname, formal title
Pronouns: she/her
Significance: 3
Identity: A concise physical or social identity.
Core: The character's stable nature and defining principles.
Personality: A few durable personality traits.
Relational disposition: How she generally approaches trust, affection, commitment, and conflict.
Protected traits: Qualities ordinary story events should not casually erase.
```

Only the correctly titled Core card is required. Keep it concise and describe durable characterization rather than the current scene.

`Significance` accepts `1` through `5` and defaults to `3`:

| Value | Suggested use |
|---:|---|
| `5` | Central companion, rival, or major recurring character |
| `4` | Important recurring character |
| `3` | Regular supporting character |
| `2` | Minor recurring character |
| `1` | Background or occasional character |

Significance breaks scheduling ties; it cannot make an off-scene NPC active. A directly involved lower-significance NPC takes priority over a higher-significance NPC elsewhere.

Use `Enabled: false` to ignore an NPC temporarily. Leave ordinary Story Card trigger keys blank for CC cards because the script discovers them by title.

## Player identity

The optional `CC PLAYER` card helps preserve player control and grammar:

```text
Name: Azure
Pronouns: she/her/her/hers/herself
```

Common two-part forms such as `she/her`, `he/him`, and `they/them` expand automatically. Creator cards may use `{{player.name}}`, `{{player.subject}}`, `{{player.object}}`, `{{player.possessive_adjective}}`, `{{player.possessive_pronoun}}`, and `{{player.reflexive}}`.

## Optional creator cards

Use the exact NPC name from the Core card:

- `CC VOICE: Full Name` — speech style, rhythm, vocabulary, and mannerisms.
- `CC MOTIVES: Full Name` — durable needs, fears, priorities, and ambitions.
- `CC BOUNDARIES: Full Name` — creator-defined limits, consent rules, and relationship boundaries.

The script selects one relevant anchor for the focused NPC when context space permits.

## How staggered focus works

The script can recognize up to five scene-active NPCs in one response. Scene presence is inferred from the current player input, Recent Story, and brief scene carryover. A clear time jump resets that carryover.

For each response, the script:

1. Identifies the registered NPCs currently involved in the scene.
2. Ranks direct involvement and current or recent presence first.
3. Uses `Significance` to prioritize a crowded active cast.
4. Chooses one focus NPC, preferring direct involvement and then the least recently served eligible NPC.
5. Supplies compact Core identity for every active NPC and deeper relevant continuity for only the focus NPC.

This keeps multiple NPCs distinct without showing the model several deep continuity packets in one response.

A player action and Continue each count as a story turn. Retry replaces the same position, repeats the same focus, restores the earlier evidence window, and cannot receive or apply a continuity update. It does not add evidence or consume the review cooldown.

## Current reactions and durable change

There is no `CC STATE` card or State output task in v5.0.1. Current reactions remain ordinary story facts. The model sees the same live sources it already uses for portrayal and infers the NPC's present state from them.

Durable persistence is deliberately slower:

1. **Observe:** A completed response focused on an NPC is saved as that NPC's generated-story evidence.
2. **Accumulate:** Later focused responses build the same owner's small evidence window.
3. **Review:** Once the owner has enough observations, a review waits until that NPC is focused again and the global review cooldown is clear.
4. **Reset:** Whether the review stores a change or finds none, that owner's reviewed window resets.

Only one owner can be reviewed in a response. If several owners are ready, each waits for a later focus turn. After a delivered review, the default cooldown guarantees at least one successful prose-only turn before another review.

Recent Story is not copied into the continuity packet. During a mature review, the script supplies only older owner evidence that is not already present in Recent Story. Evidence accumulation itself produces no metadata and uses no output tokens.

A durable review may ask the model for one short final `CC5` record after complete story prose. The record is optional: no record is the correct result when the story establishes no lasting change. The Output modifier removes valid, rejected, or malformed terminal records before prose enters history.

Player input alone is never evidence. The generated story must support the durable change. Brief emotions and immediate intentions remain in live story context rather than becoming permanent continuity.

## Configuration card

The script creates `Configure Character Continuity`. Edit values after the colon and keep the labels intact.

| Setting | Meaning |
|---|---|
| `Enabled` | `false` disables injection and updates while preserving cards. |
| `Context ceiling` | Maximum estimated tokens used by the continuity packet and an optional review task. Default: `1400`. |
| `Durable observations before review` | Focused completed responses required for an owner to mature. Default: `2`. |
| `Prose-only turns after review` | Successful turns required before another owner may review. Default: `1`. |
| `Turning Points` | `growth-only`, `all-directions`, or `disabled`. |
| `Maximum active NPCs` | Maximum scene-present NPCs included in one response. Default and maximum: `5`. |
| `Debug` | Creates an additional diagnostics card when `true`. |

The context ceiling controls Character Continuity's injection, not the model's total context window. Optional details are omitted before essential active-NPC identity. If a mature review cannot fit safely, it is deferred and the story proceeds without the task.

There is no Character Continuity output-token recommendation or low-output warning in v5.0.1. Ordinary model output limits still affect how much story the model can write, but inferred reactions and evidence accumulation do not consume output tokens. The rare durable record is short, optional, and never allowed to replace complete prose.

## Managed cards

The script creates and maintains:

- `CC CONTINUITY: Name` — durable personal continuity.
- `CC RELATIONSHIPS: Name` — durable directional relationships.
- `CC MEMORY ARCHIVE: Name` — significant durable memories.
- `CC TURNING POINTS: Name` — major durable development.
- `Character Continuity Status` — compact runtime information.
- `Character Continuity Debug` — optional diagnostics.

An obsolete `CC STATE: Name` card from an earlier v5 build is removed during cutover.

Manual edits to managed durable cards are supported. Keep field labels and complete relationship or Turning Point blocks intact. Clearing a field intentionally clears its stored value. Avoid renaming managed cards.

The first automatic model output is suppressed. The first genuine player action receives a prose-only startup generation before durable reviews may begin.

## Reading the Status card

The status card reports:

- Registered, scene-active, and focused NPCs, including significance.
- Focused evidence counts and which owners are ready.
- The current prose-only review cooldown.
- The last review owner, candidate result, and applied update.
- Continuity context use and retained Recent Story size.
- Whether the generation was a Retry, Continue, or graceful fallback.

It intentionally contains no output allowance or mode recommendation.

## Troubleshooting

**No configuration, status, or managed cards appear:** Confirm that the Library is connected to all three modifiers and at least one card is titled exactly `CC CORE: Name`.

**An NPC is not active:** Mention the NPC's name or a listed alias in current input or Recent Story. Check `Maximum active NPCs` and `Enabled`. Significance does not override scene presence.

**The same NPC remains focused:** Directly naming that NPC keeps immediate relevance ahead of fairness. In group scenes, avoid unnecessarily repeating only one name if you want the rotation to move naturally.

**An owner says `ready` but is not reviewed:** The owner must become focused again, the prose-only cooldown must be zero, and the optional review task must fit the context ceiling.

**A review stores nothing:** This is normal. The evidence may describe only a temporary reaction, repeat existing continuity, or contain no sufficiently supported lasting change.

**A CC5 line becomes visible:** Treat it as a beta bug. Save the output and Status card before retrying.

**A later output is blank:** Check the Status card. The script suppresses the initial automatic output and rejects a response that contains no complete story prose. After an incomplete response it requests ordinary prose recovery; no evidence or durable update advances.

## Short open-beta test tracks

### Track A: Multi-NPC focus

Use three to five registered NPCs in one scene for 10–15 successful turns. Mix player actions and Continue. Confirm that all active characters stay distinct while deeper focus rotates according to direct involvement, significance, and fairness.

### Track B: Durable maturation

Create one lasting decision or relationship change. Watch the owner's evidence count mature, then confirm that its later review stores only generated-story-supported continuity. It is valid for a review to store nothing.

### Track C: Retry and time jump

Retry a response near a mature review and confirm that the discarded update is rolled back and Retry receives no review task. Then make a clear time jump and confirm that old scene presence falls away while durable continuity survives.

## Reporting beta issues

Useful details include:

- Character Continuity version.
- Model and ordinary output setting.
- Relevant NPC significance values.
- Scene-active and focused NPCs from the Status card.
- Evidence counts and review cooldown.
- Whether the action was normal, Retry, or Continue.
- Whether a time jump occurred.
- The unexpected output and relevant managed card before and after.
- `Character Continuity Status` and optional `Character Continuity Debug`.

Copyable template:

```text
Version:
Model and output setting:
NPC significance values:
Normal action, Retry, or Continue:
Scene-active NPCs / focus:
Evidence counts / review cooldown:
What I expected:
What happened:
Time jump involved: Yes / No
Visible CC5 text: Yes / No
Story prose lost: Yes / No
Status card:
Relevant managed card before/after:
Other notes:
```

Private story text is not required. Redacted excerpts plus the Status card are usually enough.

The most important failures to report are lost prose, visible CC5 records, a review assigned to the wrong NPC, Retry preserving a discarded update, temporary events becoming durable, off-scene NPCs displacing active ones, or creator edits being overwritten.
