---
order: 10
route: /usage/core-concepts/worldinfo/advanced/
templating: false
---

## Vector Storage Matching

The Vector Storage extension provides an alternative to keyword matching by using the similarity between the recent chat messages and World Info entry contents.

To enable and use this, the following prerequisites need to be met:

1. Vector Storage extension is enabled and is configured to use one of the available embedding sources.
2. The "Enable for World Info" checkbox is ticked in the Vector Storage extension settings.
3. Either the World Info entries that are allowed for keyless matching have the "Vectorized" (🔗) status or the "Enabled for all entries" option is checked in the Vector Storage settings.

The choice of the vectorization model in the extension and the theoretical meaning behind the term "embeddings" won't be covered here. Check out the [Data Bank](/Usage/Characters/data-bank.md#vector-storage) guide if you require more info on this topic.

Vector Storage matching adheres to this set of rules:

* The maximum number of entries that are allowed to be matched with the Vector Storage can be adjusted with the "Max Entries" setting. This number only sets the limit and does not influence the token budget set in the activation settings for World Info. All of the budgeting rules still apply.
* This feature only replaces the check for keywords. All additional checks must be met for the entry to be inserted: trigger%, character filters, inclusion groups, etc.
* The "Scan Depth" setting from Activation Settings or entry overrides is not used. The Vector Storage "Query messages" value is utilized instead to get the text to match against. This allows for a configuration like "Scan Depth" set to 0, so no regular keyword matches will be made, but entries still can be activated by vectors.
* A "Vectorized" status is only an additional marker. The entry would still behave like a normal, enabled, non-constant record that will be activated by keywords if they are set. Remove the keywords if you want them to be activated only by vectors.

!!!info Note
Since the retrieval quality depends entirely on the outputs of the embedding model, it's impossible to predict exactly what entries will be inserted. If you want deterministic and predictable results, stick to keyword matching.
!!!

## Timed Effects

Usually, World Info evaluation is stateless, meaning that the result of the evaluation is the same, only depending on the current chat context. However, with the introduction of Timed Effects, you can create entries that have an activation delay, stay active after being triggered, or can't be triggered after the activation.

### Timed Effects Rules

1. The time frames for the effects are measured in messages (not pairs of messages/exchanges), with 0 meaning there is no effect.
2. Effects only apply in the chat where the entry was activated. Branches inherit the state of the parent chat.
3. Active timed effects are removed if the chat doesn't advance, e.g. if the last message was swiped or deleted.
4. Making any changes to the entry that is currently on timed effect will cause the effect to be forcibly removed.
5. Consequent triggering of keywords does not refresh the effect duration if it's already active.

### Types of Timed Effects

1. Sticky - the entry stays active for N messages after being activated. Stickied entries ignore probability checks on consequent scans until they expire.
2. Cooldown - the entry can't be activated for N messages after being activated. Can be used together with sticky: the entry goes on cooldown when the sticky duration ends.
3. Delay - the entry can't be activated unless there are at least N messages in the chat at the moment of evaluation.
    * Delay = 0 -> The entry can be activated at any time.
    * Delay = 1 -> The entry can't be activated if the chat is empty (no greeting).
    * Delay = 2 -> The entry can't be activated if there is zero or only one message in the chat, etc.

### Timed Effects Example

Entry configuration: sticky = 3, cooldown = 2, delay = 2.

```txt
Message 0: delay
Message 1: entry activated
Message 2: sticky
Message 3: sticky
Message 4: sticky
Message 5: cooldown
Message 6: cooldown
Message 7: entry can be activated again
```
