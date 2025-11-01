---
order: 80
icon: globe
route: /usage/core-concepts/worldinfo/global/
templating: false
---

# Global Activation Settings

These settings apply to all activation logic and may be configured from a collapsible menu located at the top of the World Info screen.

## Scan Depth

> Can be overridden on a per-entry basis.

Defines how many messages in the chat history should be scanned for World Info keys, starting from the most recent.

* When set to `0`, only recursed entries and Author's Note are evaluated.
* When set to `1`, SillyTavern only scans the last message.
* When set to `2`, the two most recent messages are considered; there is no practical upper limit.

## Context % and Budget Cap

These specify how many tokens are available for World Info entries in each prompt sent to the LLM.

This can be defined two ways: as a percentage of your configured [Context size](/Usage/Common-Settings.md#context-tokens) (Context %) and as an absolute value (Budget).

The World Info portions of the prompt will be [filled by priority](./worldinfo.md#prompt-insertion) until the available limit is reached. Once the limit is reached, no more activated entries will be considered. 

## Min Activations

*This setting is mutually exclusive with Max Recursion Steps.*

When set to a non-zero value, this will ignore the value set for [Scan Depth](#scan-depth), instead reading as much chat history as needed to reach the target minimum number of activated entries.

This setting will respect [Max Depth](#max-depth) and the [context budget](#context--and-budget-cap).

*The Min Activations pass occurs before any [recursive logic](#recursive-scanning) is evaluated. However, any entries activated by this mechanism will still be processed as usual, including their recursive steps.*

## Max Depth

The Maximum Depth to scan, used to set a limit on how many chat messages [Min Activations](#min-activations) can consider.

Its purpose is to prevent the search from going so far back as to include now-irrelevant information.

## Max Recursion Steps

*This setting is mutually exclusive with Min Activations.*

This serves as a limit on [recursive scanning](#recursive-scanning). When set to `0`, recursion is limited only by your [context budget](#context--and-budget-cap). When set to a non-zero value, this sets a limit on how many successive evaluation passes will be performed following the activation of new entries.

Example cases:

* Max = `1`: no additional checks will be performed; whatever is activated in the first pass is all that will be activated
* Max = `2`: after the first pass runs, a second pass is performed, including the original [scan-buffer](#scan-depth) and any newly activated entries
* Max = `3`: after the second pass above, a third pass is performed, including whatever entries were newly activated

## Include Names

Specifies whether the names of chat participants should be included in the text scan-buffer as message prefixes. This allows names to be matched as [Keywords](./worldinfo.md#keywords) without the name needing to be present in the message itself.

Assuming the participants are named Alice and Bob, this would have an effect as illustrated below:

Enabled (default):

```txt
Alice: Hello! Good to see you.
Bob: How is the weather today?
```

Disabled:

```txt
Hello! Good to see you.
How is the weather today?
```

## Recursive scanning

Recursive scanning enables activated entries to activate others in turn, over and over, to resolve complex relationships between World Info entries. This feature can significantly enhance the dynamic nature of creative scenarios.

When this setting is disabled, behavior is effectively the same as setting [Max Recursion Steps](#max-recursion-steps) to `1`.

See [the entry section on recursion](./worldinfo.md#recursion) for a list of options that may be adjusted on a per-entry basis.

The gist of recursion is that *entries can activate other entries by mentioning their keywords in the content text.*

For example, suppose your Lorebook contains two entries:

```txt
Entry #1
Keyword: bessie
Content: Bessie is a cow and is friends with Rufus.
```

```txt
Entry #2
Keyword: rufus
Content: Rufus is a dog.
```

Next, suppose the text `Bessie saw her friend playing in the field.` is part of your [scan-buffer](#scan-depth). When this happens, *both* entries will be activated because "Bessie" activated the `bessie` entry and it activated the `rufus` entry in a recursive pass. Now your LLM knows that Bessie's friend in the field is most likely a dog named Rufus, letting it tell a more coherent story.

## Case-sensitive keys

> Can be overridden on a per-entry basis.

This makes [Keyword](./worldinfo.md#keywords) matching more strict, requiring that words match exactly in terms of capitalization.

This is mostly useful if your keys are proper nouns within your chat, like the names of important people or cities.

It is likely quite unhelpful if your chat takes the form of text-message exchanges where the style is often intentionally lazy.

For example, when this setting is active, `rose` will *not* match `Rose` because `'r' != 'R'`.

## Match whole words

> Can be overridden on a per-entry basis.

This makes [Keyword](./worldinfo.md#keywords) matching more strict, requiring that each keyword matches a word in the input based on common word-boundary markers (spaces, periods, hyphens).

This is usually a good idea because `cat` probably shouldn't match `concatenate`.

If you want to match words that start or end with a certain sequence of characters, consider [regular expressions](./advanced.md#regular-expressions-regex-as-keys).

*Important: this setting is often incompatible with languages that don't use whitespace to separate words (e.g. Japanese, Chinese). If you write keywords in these languages, it is advised to turn it off.*

## Use Group Scoring

> Can be overridden on a per-entry basis.

This adds additional filtering logic to entries when Inclusion Groups are in use; for full details, see [the notes on Group Scoring in the entry structure](./worldinfo.md#group-scoring).

## Alert on overflow

Shows an alert when enough entries are activated that their size exceeds the [context budget](#context--and-budget-cap).

Entries will still be discarded as necessary; this just makes it more obvious when experimenting with World Info.

## Insertion Strategy

This selects a prioritization approach for sources of World Info.

### Sorted Evenly <sub>(default)</sub>

All entries will be sorted according to their [Insertion Order](./worldinfo.md#prompt-insertion) as if they were part of a single large World Info source, regardless of how they are actually structured.

#### Character Lore First

Entries from character- and persona-bound World Info are included first, then any remaining [context budget](#context--and-budget-cap) is allocated to global- and chat-bound World Info.

#### Global Lore First

Entries from global- and chat-bound World Info are included first, then any remaining [context budget](#context--and-budget-cap) is allocated to character- and persona-bound World Info.
