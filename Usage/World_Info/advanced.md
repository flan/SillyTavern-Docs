---
order: 10
route: /usage/core-concepts/worldinfo/advanced/
templating: false
---

# Advanced Interactions

## Regular Expressions (regex) as keys

Both [Primary Keywords](./worldinfo.md#primary-keywords) and [Optional Filter](./worldinfo.md#optional-filter) support [regular expression syntax that complies with the Mozilla-published specification](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions).

Regular expressions allow a single string to match multiple possible pieces of text, reducing the amount of repetition and prediction you need to account for.

Consider a bit of text like "Computer, switch to DefProt Red 7." Suppose that, in your world, you have a lot of numbered, colored defense-protocols, and that `DefProt` on its own doesn't refer to a specific battle-plan. You *could* write a long list of keywords like `defprot red 1,defprot orange 2,defprot yellow 1`, but that's tedious to read and maintain; you could also match on `defprot` `AND ANY` `red,green,orange`, but then that might match someone talking about fruit and it doesn't account for the number.

With regular expressions, you could reduce this to `/defprot (red|yellow|orange|banana) \d+/i` and solve all of the problems at once.

They can be combined with macros, too, for even more specialized cases:

```javascript
/({{char}}|he|she) (is talking about|is noticing|is checking whether|observes) (the )?(rainy weather|heavy wind|it is going to rain|cloudy sky)/i
```

This would create an entry that reacts to the current character doing something specific to the weather.

### Matching on a per-character basis

The text considered for World Info matching, the scan-buffer, is prefixed with `{{char}}:` or `{{user}}:` before every message when the [Include Names](./global.md#include-names) option is set.

Starting with SillyTavern version v1.12.6, the scan-buffer also delimits messages with the non-printable character `\x01`. These two properties together allow a regular expression to look for text produced by a specific character, which can help to gate information in a Lorebook based on a sort of ownership.

For example, the following regex would trigger only when the user says "banana bread" somewhere in their message:

```javascript
/\x01{{user}}:[^\x01]*?.*?\bbanana bread\b.*?[^\x01]*?/i
```

*Note: the `[^\x01]` parts avoid having the search creep into another character's messages.*

## Timed Effects

World Info evaluation is usually stateless, meaning that an entry is either activated or not, with this being determined solely by the current contents of the chat.

Timed Effects present another layer to World Info entries, however: they may have an activation delay, stay active after being triggered, or become temporarily unusable after being activated. This allows them to have longer-term effects on your chat with very little additional setup.

### Timed Effects Rules

1. Time-frames for Effects are measured in terms of total messages, both those written by the user and by any characters in the chat.
1. Effects only apply within the chat where the entry was activated; branches inherit the state of the parent chat.
1. Newly activated Timed Effects are reset if the chat does not advance.
   - For example, if the message that activated the entry is swiped or deleted, the entry is no longer considered active.
1. If an entry that triggered a Timed Effect is modified, the Timed Effect will be removed.
1. A Timed Effect that is already active cannot be refreshed by triggering it again.

### Types of Timed Effects

- **Sticky**
  - The entry stays active for `n` messages after being activated.
  - Stickied entries ignore activation checks, like trigger %, on consequent scans until they expire.
- **Cooldown**
  - The entry cannot be re-activated for `n` messages after being activated.
  - Can be used together with sticky Effects: the entry goes on cooldown when the sticky duration ends.
- **Delay**
  - The entry cannot be activated unless there are at least `n` messages in the chat at the time of evaluation.
    - Delay = `0` -> the entry can be activated.
    - Delay = `1` -> the entry cannot be activated if the chat is empty (a greeting counts as a message).
    - Delay = `2` -> the entry cannot be activated if there are fewer than `2` messages in the chat.
    - ...And so forth, with no effective limit.

### Timed Effects Example

Entry configuration: sticky = `3`, cooldown = `2`, delay = `2`.

- Message `0`: delay
- Message `1`: entry activated
- Message `2`: sticky
- Message `3`: sticky
- Message `4`: sticky
- Message `5`: cooldown
- Message `6`: cooldown
- Message `7`: entry can be activated again

## Vector Storage Matching

The [Vector Storage extension](../../extensions/Chat-vectorization.md) provides an alternative to keyword matching by using the similarity between recent chat messages and World Info entry contents.

### Pre-requisites
To enable and use Vector Storage matching, some setup is required:

1. The Vector Storage extension must be enabled and configured to use an available embedding source.
2. The "Enable for World Info" checkbox must be ticked in the Vector Storage extension settings.
3. Either the World Info entries that are allowed for keyless matching must have the "Vectorized" (🔗) status or the "Enabled for all entries" option must be checked in Vector Storage settings.

The choice of the vectorization model in the extension and the theoretical meaning behind the term "embeddings" won't be covered here. Check out the [Data Bank](/Usage/Characters/data-bank.md#vector-storage) guide if you require more information on this topic.

### Behavior
Vector Storage matching adheres to these rules:

1. The maximum number of entries that are allowed to be matched with Vector Storage can be adjusted with the "Max Entries" setting.
   - This number only sets the limit and does not influence the token budget specified in the insertion settings for World Info.
   - All budgeting rules still apply.
1. This feature replaces [Keywords](./worldinfo.md#keywords) and *only* Keywords.
   - All other checks must be satisfied for the entry to be activated: trigger %, character filters, inclusion groups, etc.
1. The [Scan Depth](./worldinfo.md#scan-depth) setting is not used.
   - The Vector Storage "Query messages" value is used instead to select the text for comparison.
   - This allows and encourages Scan Depth to be set to `0`, which will prevent Keyword matches from being possible, while still allowing entries to be activated.
1. The "Vectorized" (🔗 (Chain Link)) status does not confer any special activation priority.
   - A Vectorized entry behaves the same as a normal (🟢 (Green Circle)) entry in terms of both activation and [Prompt Insertion](./worldinfo.md#prompt-insertion).
   - A Vectorized entry may still be activated by Keywords if any are set and Scan Depth is non-zero or recursion might apply.

*Note: the effectiveness of retrieval depends entirely on the outputs of the embedding model. It is impossible to predict exactly which entries will be activated. If you want deterministic and predictable results, stick to keyword matching.*
