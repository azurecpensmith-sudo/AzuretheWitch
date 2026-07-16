# Character Continuity v5.1.2 Open Beta

## Thought Triggers and Reliable Persistence — Creator Guide

Character Continuity helps AI Dungeon portray recurring NPCs consistently while allowing temporary inner state, lasting decisions, relationships, memories, and development to survive beyond the immediate scene.

v5.1.2 combines four layers:

- **Staggered portrayal:** up to five scene-active NPCs receive individual guidance while one receives deeper focus.
- **Thought-trigger State:** the model writes one private, first-person Thought and selects controlled triggers; the script derives compact Feeling, Goal, Tension, and temporary relational pressure.
- **Durable persistence:** stricter periodic reviews may record lasting personal or relational change.
- **Opt-in onboarding:** queued NPCs can receive editable Core and Voice cards from supported story context.

It also detects Retry from replacement of the last visible output, even when AI Dungeon does not rerun the Input modifier.

### What v5.1.2 changes

v5.1.2 replaces the model-authored four-field State snapshot with a smaller and more reliable division of work:

- The model supplies subtext that requires interpretation: one first-person private Thought, a target, and one to three controlled triggers.
- The script deterministically derives Feeling, Goal, Tension, and temporary trust, closeness, or conflict pressure.
- Leading, inline, and terminal hidden records are normalized safely instead of making harmless placement variation fatal.
- A malformed Thought does not discard valid trigger-derived State.
- Direct interaction outranks incidental name mentions for focus, and absent focus owners no longer gain evidence.
- A durable review that returns no update retains its evidence, waits for one new owner-specific observation, and may review again later.
- Retry-generated hidden-record debris is suppressed without changing continuity or producing a false runtime error.

This is an open beta. Test it in a copy of an adventure and back up important cards before updating.

> **Compatibility:** Character Continuity requires the Context modifier. Cached models and Optimized Context are unsupported.

## Quick setup

1. Add the Character Continuity v5.1.2 script as a Library.
2. Connect it once to each Input, Context, and Output modifier.
3. Create `CC CORE: Full Name` for NPCs known at scenario start.
4. For NPCs to be discovered later, add their names to `CC NPC QUEUE` instead.
5. Optionally create `CC PLAYER`, `CC VOICE`, `CC MOTIVES`, and `CC BOUNDARIES` cards.
6. Start the adventure. CC creates its configuration, status, State, continuity, relationship, memory, and Turning Point cards.

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

## Registering an NPC manually

Create a Story Card titled exactly:

```text
CC CORE: Full Name
```

Recommended format:

```text
Enabled: true
Aliases: Nickname, formal title
Pronouns: she/her
Significance: 3
Identity: A concise physical or social identity.
Core: The character's stable nature and defining principles.
Personality: A few durable personality traits.
Relational disposition: How she generally approaches trust, affection, commitment, and conflict.
Protected traits: Qualities ordinary story events should not casually erase.
```

Keep Core cards concise and durable. Current feelings and scene-specific intentions belong in Temporary State.

`Significance` accepts `1` through `5` and defaults to `3`:

| Value | Suggested use |
|---:|---|
| `5` | Central companion, rival, or major recurring character |
| `4` | Important recurring character |
| `3` | Regular supporting character |
| `2` | Minor recurring character |
| `1` | Background or occasional character |

Significance breaks scheduling ties; it cannot make an off-scene NPC active. Use `Enabled: false` to ignore an NPC temporarily. Leave ordinary Story Card trigger keys blank because CC discovers its cards by title.

## Discovering NPCs during play

CC never auto-registers every named character. Onboarding is opt-in through the player-editable `CC NPC QUEUE` card.

Add one unchecked NPC per line:

```text
- [ ] Mira Vale
```

Optional details may be supplied after the name:

```text
- [ ] Mira Vale | Significance=4 | Pronouns=she/her | Notes=travelling surveyor
```

The flow is:

1. CC waits until the exact queued name appears in current input or Recent Story.
2. On an eligible generation, the model may return one compact hidden registration snapshot as the first nonblank output line, based only on available story context.
3. Complete story prose must also survive. A registration-only response creates nothing and triggers prose recovery.
4. CC creates an editable `CC CORE: Name`, an optional `CC VOICE: Name`, and the normal managed cards.
5. The queue line changes to `[x]`, and the NPC can become scene-active on the following generation.

Generated Core cards begin with:

```text
Review: pending
```

Review and correct the profile whenever the first encounter was misleading. Creator edits are authoritative; CC does not regenerate or overwrite the Core or Voice profile.

If the registration output is retried, unchanged auto-created cards are removed and the queue entry is reopened. If any generated profile card has already been edited, CC preserves the corrected registration instead of deleting the player's work.

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

## Staggered focus

CC recognizes up to five scene-active NPCs from current input, Recent Story, and brief scene carryover. A clear time jump resets that carryover.

For each response, CC:

1. Finds registered NPCs involved in the scene.
2. Scores direct address and interaction more strongly than incidental name mentions, then considers current or recent presence and Significance.
3. Selects one focus NPC, preferring the strongest direct involvement and then the least recently served eligible NPC.
4. Supplies compact identity and any live State for every active NPC.
5. Supplies deeper relevant relationships, memories, motives, boundaries, or Turning Points only for the focus NPC.

Player actions and Continue both participate in focus rotation. Retry repeats the discarded turn's active cast and focus.

## Thought-trigger Temporary State

Each managed `CC STATE: Name` card can contain:

```text
Feeling: nostalgic
Thought: I hope she remembers the good parts, not just the creaky floorboards.
Goal: Revisit shared history
Tension:
Triggers: nostalgia, reconnect
Target: Azure
Relational pressure: Azure — closeness +1
```

The model no longer writes all four State fields. On an eligible focused turn, it may return one compact hidden operation containing:

- one short first-person private `Thought` in the NPC's own voice;
- one target: self, player, or another scene-active NPC;
- one to three controlled trigger codes.

The script validates and stores the Thought, then deterministically derives Feeling, Goal, Tension, and any temporary relational pressure from the triggers. Grounded private subtext may be inferred from Core, Voice, relationships, current input, and Recent Story, but it cannot invent an event or control the player.

The controlled vocabulary is:

| Group | Trigger codes |
|---|---|
| Affect | `affection`, `anger`, `anxiety`, `curiosity`, `grief`, `hope`, `hurt`, `relief`, `resolve`, `uncertainty` |
| Direction | `nostalgia`, `protect`, `reconnect`, `withdraw`, `responsibility`, `conflict` |
| Relationship | `trust_gain`, `trust_loss`, `closeness_gain`, `closeness_loss`, `boundary_respected`, `boundary_crossed`, `repair`, `betrayal` |

The strongest supported trigger comes first. Unknown triggers are ignored, and only the first three valid triggers are used. The script's mapping is fixed: for example, `nostalgia` derives a nostalgic Feeling and a Goal to revisit shared history; `boundary_crossed` derives an unsafe and angry Feeling, a Goal to reassert the boundary, and temporary trust/conflict pressure.

Relational pressure is working continuity, not a durable trust score. It is capped, ages out with State, clears on a time jump, and may only make a durable relationship review eligible when at least one owner-specific generated-story observation also exists. The generated story must still support any lasting update.

State rules:

- Complete story is mandatory; metadata can never replace prose.
- One fixed focus owner may update per generation.
- No State snapshot is requested during startup grace, Retry, prose recovery, durable review, or NPC registration.
- The operation updates the whole trigger-derived working snapshot. Existing State remains unchanged when no operation is returned.
- A missing or invalid first-person Thought is ignored without discarding otherwise valid trigger-derived fields.
- State cannot decide the player's actions, feelings, consent, commitments, or relationship response.
- Each stored field and relational pressure retains an update clock and expires after the configured number of successful non-Retry generations unless refreshed.
- Explicit time jumps clear prior temporary State and relational pressure.
- Retry restores the pre-generation State and pressure and neither updates nor ages them.

Manual edits to `CC STATE` are supported. A populated creator-made State card is imported before CC's first managed write instead of being overwritten. Keep the field labels intact.

## Durable persistence

Temporary reactions do not automatically become lasting continuity. Durable persistence remains stricter:

1. **Observe:** a completed response focused on an NPC becomes that owner's generated-story evidence only when that NPC is actually portrayed.
2. **Accumulate:** later focused responses build a small evidence window.
3. **Schedule:** enough observations normally mature a review. Strong temporary trust, closeness, or conflict pressure may schedule it earlier, but never without at least one owner-specific observation.
4. **Review:** the review waits until that NPC is focused and the global cooldown is clear.
5. **Resolve:** an accepted update clears reviewed evidence and matching pressure. An absent or rejected update retains the evidence, records the attempt, and waits for one new owner-specific observation before reviewing again.

Only one owner can receive a durable review in a response. The model may append one short hidden record as the last nonblank output line after complete story, but no record is correct when no lasting change was established.

Player input and relational pressure are never durable evidence. Generated story must support the stored change. Newly revealed lasting memories, explicit acceptance or refusal of an ongoing responsibility, established boundaries, and supported relationship changes can all qualify. A temporary mood, inferred private Thought, or pressure value alone cannot.

## Retry and revisions

v5.1.2 saves a fingerprint of each complete visible output and its pre-generation continuity snapshot. At the next Context call:

- If the prior output still exists unchanged, generation proceeds normally.
- If it was removed or changed, CC restores the snapshot and treats the generation as Retry or revision.

Retry:

- repeats the original focus and active cast;
- occupies the discarded story-turn position rather than adding a new one;
- cannot request or apply State, durable persistence, or NPC registration;
- requests complete story only; if the model or host repeats discarded hidden-record debris anyway, CC strips and suppresses it without treating it as a continuity change or runtime error;
- cannot add durable evidence, consume cooldown, or age State;
- retains the same rollback anchor across consecutive Retries.

Complete replacement prose still survives normally.

## Configuration card

Edit values after the colon in `Configure Character Continuity` and keep the labels intact.

| Setting | Meaning |
|---|---|
| `Enabled` | `false` disables injection and updates while preserving cards. |
| `Context ceiling` | Maximum estimated tokens used by the continuity packet and optional task. Default: `1400`. |
| `Temporary State lifetime` | Successful non-Retry generations before an unrefreshed State field or relational pressure expires. Default: `3`. |
| `Durable observations before review` | Focused completed responses required for an owner to mature. Default: `2`. |
| `Prose-only turns after review` | Successful turns required before another owner may review. Default: `1`. |
| `Turning Points` | `growth-only`, `all-directions`, or `disabled`. |
| `Maximum active NPCs` | Maximum scene-present NPCs included per response. Default and maximum: `5`. |
| `Debug` | Creates an additional diagnostics card when `true`. |

The ceiling applies only to CC's injection, not the model's total context window. Optional details are omitted before essential active-NPC identity. If an optional State, registration, or durable task cannot fit safely, it is deferred and story proceeds.

There is no hard output-token minimum. Thought-trigger and registration operations consume some output space when returned, though the Thought-trigger operation is smaller than the old four-field snapshot. At very short output settings, complete prose remains protected and a missing operation leaves existing continuity unchanged. If operations are repeatedly absent, test a longer ordinary output setting before reporting a persistence failure.

## Managed cards

CC creates and maintains:

- `CC STATE: Name` — expiring Thought-trigger State plus temporary relational pressure.
- `CC CONTINUITY: Name` — durable personal continuity.
- `CC RELATIONSHIPS: Name` — durable directional relationships.
- `CC MEMORY ARCHIVE: Name` — significant durable memories.
- `CC TURNING POINTS: Name` — major durable development.
- `Character Continuity Status` — compact runtime information.
- `Character Continuity Debug` — optional diagnostics.

Manual edits to managed cards are supported. Keep field labels and complete relationship or Turning Point blocks intact. Avoid renaming managed cards.

The first automatic model output is suppressed. The first genuine player action receives a prose-only startup generation before hidden State, registration, or durable tasks may begin.

## Reading the Status card

The Status card reports:

- Registered, queued, scene-active, and focused NPCs.
- Which State fields and trigger codes exist for each NPC, the State clock/lifetime, and temporary trust/closeness/conflict pressure.
- Durable evidence counts and review readiness.
- Review cooldown and the last task owner.
- Candidate and update results.
- Context use by rules, NPC identity, State, relationships, memories, and task.
- Retry, Continue, fallback, and time-jump handling.

When Debug is enabled, it also reports whether the focus owner was admitted as generated-story evidence and the exact normalization or rejection reason for the most recent hidden record.

## Troubleshooting

**No configuration or status card appears:** Confirm that the script is connected once to all three modifiers.

**A queued NPC is not created:** The unchecked exact name must appear in current input or Recent Story, startup grace must be complete, and the registration task must fit. Thin evidence may correctly leave the NPC queued.

**A generated NPC is inaccurate:** Edit `CC CORE: Name` or `CC VOICE: Name`. Leave `Review: pending` until satisfied. CC will not overwrite the correction.

**An NPC is not scene-active:** Mention the registered name or an alias in current input or Recent Story. Check `Maximum active NPCs` and `Enabled`.

**State is empty:** Thought-trigger updates are optional and owner-bound. The model may find no supported change, State may have expired, or the turn may have been reserved for registration or durable review.

**Thought was rejected:** It must be explicitly first person and written as the NPC's private thought. In v5.1.2, valid triggers may still update derived Feeling, Goal, Tension, or pressure even when only the Thought field is rejected.

**The wrong NPC was focused:** Direct address and interaction cues should outrank an incidental mention. Save the action, output, Status, and Debug cards if they do not. Fair rotation is used after direct relevance is considered.

**Evidence did not advance:** A focus assignment alone is insufficient. The completed story must actually portray that owner. This prevents a response about another NPC or only the environment from maturing the wrong character.

**An owner is `ready` but not reviewed:** The owner must be focused, cooldown must be zero, and the review task must fit.

**A review stores nothing:** This is normal when evidence describes only a temporary reaction, repeats continuity, or establishes no lasting change. v5.1.2 retains the evidence but requires one new owner-specific observation before another review, preventing an immediate review loop while allowing a later supported change to persist.

**Status says `malformed stripped`:** The model returned a structurally invalid or unauthorized hidden record. Leading, inline, and terminal placement variations are normally normalized in v5.1.2. Complete prose survives, but an invalid operation does not change continuity. Debug includes the precise reason and captured record.

**Status says `suppressed on Retry`:** This is expected containment when the model or host repeats hidden-record debris from the discarded generation. CC strips it and leaves continuity unchanged.

**A CC5 line becomes visible:** Treat it as a beta bug. Save the complete output and Status card before retrying.

**A later output is blank:** The first automatic output is intentionally suppressed. Otherwise, CC found no complete story after stripping metadata and armed prose recovery; no continuity update advanced.

## Suggested open-beta tracks

### Track A: Thought-trigger State

Use two or three active NPCs. Confirm that direct interaction selects the correct focus, Thoughts remain first-person and character-specific, triggers derive sensible Feeling/Goal/Tension, relational pressure targets the correct character, and all temporary data expires.

### Track B: Durable reliability

Exercise several lasting change types: a decision or responsibility, a newly revealed memory, a boundary, and a relationship change. Confirm that owner-specific evidence matures and the later review stores only generated-story-supported continuity.

Then force one mature review to return no record. Confirm its evidence remains, no immediate review repeats, one new owner-specific portrayal makes a later review eligible, and a supported update can then persist.

### Track C: Relational pressure

Build repeated supported `trust_gain`, `trust_loss`, closeness, or conflict triggers toward the same target. Confirm pressure remains temporary, may schedule a review only alongside generated evidence, never becomes a durable update by itself, and clears after a matching accepted update or expiry.

### Track D: Retry and consecutive Retry

Retry a response that created State or a durable update. Confirm focus repeats, discarded continuity disappears, turn count does not gain an extra position, and a second Retry restores the same baseline again.

If Retry output repeats a hidden record, confirm the record is stripped, Status reports suppression rather than an error, and no State, pressure, evidence, cooldown, or card changes occur.

### Track E: NPC onboarding

Begin with no Core cards, queue one name, and encounter that NPC. Confirm the review-pending Core is editable and activates next generation. Retry the registration once unchanged, then repeat after editing the generated Core to test both deletion and preservation paths.

### Track F: Time jump

Create temporary State and relational pressure, make a clear multi-day or longer jump, and confirm both temporary layers and scene carryover clear while durable continuity remains.

## Reporting beta issues

Useful details include:

- Character Continuity version.
- Model and ordinary output setting.
- Relevant Core or queue entry.
- Scene-active NPCs and focus.
- State fields, triggers, target, relational pressure, evidence counts, and cooldown.
- Debug evidence-admission and CC5 parse lines.
- Normal action, Retry, Continue, or registration.
- Expected and actual story/card result.
- Complete Status card and Debug card when enabled.

Copyable template:

```text
Version:
Model and output setting:
Normal action, Retry, Continue, or registration:
Queued / scene-active NPCs / focus:
State fields:
State triggers / target / pressure:
Evidence counts / review cooldown:
Debug evidence admission / CC5 parse:
What I expected:
What happened:
Time jump involved: Yes / No
Visible CC5 text: Yes / No
Story prose lost: Yes / No
Status card:
Relevant card before/after:
Other notes:
```

The most important failures are lost prose, visible CC5 records, wrong focus after direct interaction, absent owners gaining evidence, valid State repeatedly failing to apply, supported lasting changes repeatedly failing to persist, Retry preserving discarded continuity, Retry deleting a player-edited generated profile, stale State or pressure surviving a time jump, temporary pressure becoming durable by itself, or creator edits being overwritten.
