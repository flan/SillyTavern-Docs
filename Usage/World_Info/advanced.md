---
order: 10
route: /usage/core-concepts/worldinfo/advanced/
templating: false
---

# Advanced Interactions

## Vector Storage Matching

The [Vector Storage extension](../../extensions/Chat-vectorization.md) provides an alternative to keyword matching by using the similarity between recent chat messages and World Info entry contents.

### Pre-requisites
To enable and use vectorized matching, some setup is required:

1. Vector Storage extension is enabled and configured to use an available embedding source.
2. The "Enable for World Info" checkbox is ticked in the Vector Storage extension settings.
3. Either the World Info entries that are allowed for keyless matching have the "Vectorized" (🔗) status or the "Enabled for all entries" option is checked in the Vector Storage settings.

The choice of the vectorization model in the extension and the theoretical meaning behind the term "embeddings" won't be covered here. Check out the [Data Bank](/Usage/Characters/data-bank.md#vector-storage) guide if you require more information on this topic.

### Behavior
Vector Storage matching adheres to these rules:

1. The maximum number of entries that are allowed to be matched with the Vector Storage can be adjusted with the "Max Entries" setting.
   - This number only sets the limit and does not influence the token budget specified in the activation settings for World Info.
   - All of the budgeting rules still apply.
1. This feature replaces [Keywords](./structure.md#keywords) and *only* Keywords.
   - All other checks must be satisfied for the entry to be activated: trigger %, character filters, inclusion groups, etc.
1. The [Scan Depth](./structure.md#scan-depth) setting is not used.
  - The Vector Storage "Query messages" value is used instead to select the text for comparison.
  - This allows and encourages Scan Depth to be set to `0`, which will prevent Keyword matches from being possible, while still allowing entries to be activated.
1. The "Vectorized" (🔗 (Chain Link)) status does not confer any special activation priority.
  - A Vectorized entry behaves the same as a normal (🟢 (Green Circle)) entry in terms of both [Prompt Insertion](./insertion.md) and activation.
  - A Vectorized entry may still be activated by keywords if any are set and Scan Depth is non-zero or recursion might apply.

*Note: the effectiveness of retrieval depends entirely on the outputs of the embedding model. It is impossible to predict exactly which entries will be activated. If you want deterministic and predictable results, stick to keyword matching.*

## Timed Effects

World Info evaluation is usually stateless, meaning that an entry is either activated or not, with this being determined solely by the current contents of the chat.

Timed Effects present another layer to World Info entries, however: they may have an activation delay, stay active after being triggered, or become temporarily unusable after being activated. This allows them to have longer-term effects on your chat with very little additional setup.

### Timed Effects Rules

1. Time-frames for Effects are measured in terms of total messages, both those written by the user and by any characters in the chat.
1. Effects only apply within the chat where the entry was activated; branches inherit the state of the parent chat.
1. Newly activated Timed Effects are reset if the chat does not advance
   - For example, if the message that activated the entry is swiped or deleted, the entry is no longer considered active
1. If an entry that triggered a Timed Effect is modified, the Timed Effect will be removed.
1. A Timed Effect that is already active cannot be refreshed by triggering it again.

### Types of Timed Effects

- Sticky
  - The entry stays active for `n` messages after being activated.
  - Stickied entries ignore probability checks on consequent scans until they expire.
- Cooldown
  - The entry cannot be re-activated for `n` messages after being activated.
  - Can be used together with sticky Effects: the entry goes on cooldown when the sticky duration ends.
- Delay
  - The entry cannot be activated unless there are at least `n` messages in the chat at the time of evaluation.
    - Delay = `0` -> The entry can be activated.
    - Delay = `1` -> The entry cannot be activated if the chat is empty (a greeting counts as a message).
    - Delay = `2` -> The entry cannot be activated if there are fewer than `2` messages in the chat.
    - And so forth, with no effective limit.

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
