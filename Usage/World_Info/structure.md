---
order: 100
icon: pencil
route: /usage/core-concepts/worldinfo/structure/
templating: false
---

# Lorebook Structure

At the most basic level, Lorebooks are like encyclopedia entries: there is an article and there are a few keywords that can be used to find it.

Within SillyTavern, there is some additional metadata, like how to interpret those keywords and how important each entry is, but these will make sense as you start building.

## Lorebook Entries

A Lorebook is a list of entries, and the order in which they appear is entirely for your benefit as the author: group them by purpose, sort them alphabetically... Do whatever makes sense to you.

### Anatomy of an entry

Every entry has a number of fields, presented here in the order they appear with v1.13.5.

In the first row of every entry, there is an unlabeled "enabled" toggle that lets the entry be quickly turned off entirely. This is useful for testing. and quick-swapping of state.

#### Title/Memo
Descriptive text to help the reader know what this entry is for. It is *not* sent to the LLM.

#### Strategy
- 🔵 (Blue Circle) = This entry will always be activated.
- 🟢 (Green Circle) = This entry will be activated only when its keywords are matched.
- 🔗 (Chain Link) = This entry will be matched based on [special magic math](./advanced.md#vector-storage-matching).
  - The idea here is to let the LLM score the entry according to its own understanding of language, then compare those scores to the context being evaluated to determine whether they are sufficiently similar to warrant inclusion.
  - This is a common cause of confusion when writing Lorebooks and it is not recommended while you are still learning how to interact with your LLM model, SillyTavern, and story-generation in general.

#### Position

A means of ordering where the entry will appear in the prompt sent to the LLM; see [Insertion Position](./insertion.md#insertion-position).

##### Depth
A means of determining where entries go related to each other at the same Position.

##### Order
A means of more-granularly sorting the position of entries within the final prompt.

#### Trigger %
The probability that the entry will be activated when all selection criteria are met, used to introduce some randomness.

Example uses of this could be to change the weather or to have a 1% chance of waking an Elder God with each message if your story needs a looming high-stakes threat.

#### Keywords

These are the main matching rules for Lorebook Entries.

##### Primary Keywords
A comma-delimited list of strings (text) that are used to match this entry for activation.

Example: `cat,kitten, feline` (whitespace is trimmed).

##### Optional Filter
A comma-delimited list of strings (text) that are used to further match this entry for activation.

Example: `dog, puppy,canine` (whitespace is trimmed).

##### Logic
When an Optional Filter is specified, this will determine how to interpret it using common Boolean logic:
  - `AND ANY` activates the entry only if one of the Primary Keywords and *any* of the Optional Filter keywords are present.
  - `AND ALL`: activates the entry only if one of the Primary Keywords and *all* of the Optional Filter keywords are present.
  - `NOT ANY`: activates the entry only if one of the Primary Keywords and *none* of the Optional Filter keywords are present.
  - `NOT ALL`: activates the entry only if one of the Primary Keywords and *all* of the Optional Filter keywords are present.
    - If the Optional Filter is `dog, puppy, canine` and both `dog` and `puppy` are present, but `canine` is absent, the entry will be activated.

#### Scan Depth

#### Case-Sensitive

#### Whole Words

#### Group Scoring

#### Automation ID

#### Content
    - Non-recursable
    - Delay until recursion
    - Prevent further recursion
    - Ignore budget

#### Inclusion Group
    - Prioritize

#### Group Weight
#### Sticky
#### Cooldown
#### Delay
#### Filter to Characters or Tags
    - Exclude
#### Filter to Generation Triggers

#### Additional Matching Sources

By default, entries are only evaluated against the scan-buffer specified by [Scan Depth](#scan-depth).

These options allow other sources of text to be considered, which may be useful when you want to include information that is not part of the chat-history itself or may even be descriptive metadata.

- Character Description
  - Includes `{{char}}`'s Description text.
- Character Personality
  - Includes `{{char}}`'s Personality text, located under Advanced Definitions.
- Scenario
  - Includes `{{char}}`'s Scenario text, located under Advanced Definitions.
- Persona Description
  - Includes `{{user}}`'s Description text.
- Character's Note
  - Includes `{{char}}`'s Character Note text, located under Advanced Definitions.
- Creator's Notes
  - Includes `{{char}}`'s Creator's Notes text, located under Advanced Definitions; this is typically entirely commentary.

### Regular Expressions (regex) as keys

Both Primary Keywords and Optional Filter support [regular expression syntax that complies with the Mozilla-published specification](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions).

Regular expressions allow a single string to match multiple possible pieces of text, reducing the amount of repetition and prediction you need to account for.

Consider a bit of text like "Computer, switch to DefProt 7 Red". Suppose that, in your world, you have a lot of colored defense-protocols, and that `DefProt` on its own doesn't refer to a specific battle-plan. You *could* write a long list of keywords like `defprot 1 red,defprot 2 orange,defprot 1 yellow`, but that's tedious to read and maintain; you could also match on `defprot` `AND ANY` `red,green,orange`, but then that might match someone talking about some fruit.

With regular expressions, you could reduce this to `defprot \d+ (?:red|yellow|orange|banana)` and solve all of the problems at once.

They can be combined with macros, too, for even more specialised cases:

```javascript
/(?:{{char}}|he|she) (?:is talking about|is noticing|is checking whether|observes) (?:the )?(rainy weather|heavy wind|it is going to rain|cloudy sky)/i
```

This would create an entry that reacts to the current character doing something specific to the weather.

#### Matching on a per-character basis

The text considered for World Info matching, the scan-buffer, is prefixed with `{{char}}:` or `{{user}}:` before every message. Starting with SillyTavern version v1.12.6, the scan-buffer also delimits messages with the non-printable character `\x01`. These two properties together allow a regular expression to look for text produced by a specific character, which can help to gate information in a Lorebook based on a sort of ownership.

For example, the following regex would trigger only when the user says "banana bread" somewhere in their message:

```javascript
/\x01{{user}}:[^\x01]*?.*?\bbanana bread\b.*?[^\x01]*?/i
```

(The `[^\x01]` parts avoid considering more than one message)




## Activation Settings

Collapsible menu at the top of the World Info screen.

### Scan Depth

> Can be overridden on an entry level.

Defines how many messages in the chat history should be scanned for World Info keys.

* If set to 0, then only recursed entries and Author's Note are evaluated.
* If set to 1, then SillyTavern only scans the last message.
* 2 = two last messages, etc.

### Include Names

Defines if the names of the chat participants should be included in the scanned text buffer as message prefixes. This allows activating entries that use names as keywords without directly mentioning the names in messages.

See an example of the text to be scanned below, assuming the chat participants are named Alice and Bob.

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

### Context % / Budget

**Defines how many tokens could be used by World Info entries at once.**
You can define a threshold relative to your API's max-context settings (Context %) or an objective token threshold (Budget)

If the budget is exhausted, then no more entries are activated even if the keys are present in the prompt.

Constant entries will be inserted first. Then entries with larger order numbers.

Entries inserted by directly mentioning their keys have higher priority than those that were mentioned in other entries' contents.

### Min Activations

**This setting is mutually exclusive with Max Recursion Steps.**

Minimum Activations: If set to a non-zero value, this will disregard the limitation of "scan-depth", seeking all of the chat log backward from the latest message for keywords until as many entries as specified in min activations have been triggered. This will still be limited by the Max Depth setting or your overall Budget cap.

*Additional scan sweeps triggered by Min Activations will not check entries added by recursion on previous steps. Only chat messages and extension prompts can trigger these additional activations. However, the entries activated by Min Activations can trigger other entries as usual.*

### Max Depth

Maximum Depth to scan for when using the Min Activations setting.

### Recursive scanning

Recursive scanning allows for entries to activate other entries or be activated by others, enabling complex interactions and dependencies between different World Info entries. This feature can significantly enhance the dynamic nature of your creative scenarios.  
Whether recursive scanning is enabled can be controlled with the global setting **Recursive Scan**.  
There are three options available to control recursion for each entry:

8 **Non-recursable**: When this checkbox is selected, the entry will not be activated by other entries. This is useful for static information that should not change or be influenced by other world info entries.
  
* **Prevent further recursion**: Selecting this option ensures that once this entry is activated, it will not trigger any other entries. This is helpful to avoid unintended chains of activations.

* **Delay until recursion**: This entry will only be activated during recursive checks, meaning it won't be triggered in the initial pass but can be activated by other entries that have recursion enabled. Now, with the added **Recursion Level** for those delays, entries are grouped by levels. Initially, only the first level (smallest number) will match. Once no matches are found, the next level becomes eligible for matching, repeating the process until all levels are checked. This allows for more control over how and when deeper layers of information are revealed during recursion, especially in combination with criteria as NOT ANY or NOT ALL combination of key matches.

**Entries can activate other entries by mentioning their keywords in the content text.**

For example, if your World Info contains two entries:

```txt
Entry #1
Keyword: Bessie
Content: Bessie is a cow and is friends with Rufus.
```

```txt
Entry #2
Keyword: Rufus
Content: Rufus is a dog.
```

**Both** of them will be pulled into the context if the message text mentions **just Bessie**.

### Max Recursion Steps

**This setting is mutually exclusive with Min Activations.**

When set to zero, recursion nesting is only limited by your prompt budget. When set to a non-zero value, limits the total number of scan sweeps to desired maximum "nesting level".

Example values:

* 1 effectively disables recursion as the check stops after the first step.
* 2 can only activate recursive entries once.
* 3 can trigger recursion twice...

### Case-sensitive keys

> Can be overridden on an entry level.

**To get pulled into the context, entry keys need to match the case as they are defined in the World Info entry.**

This is useful when your keys are common words or parts of common words.

For example, when this setting is active, keys 'rose' and 'Rose' will be treated differently, depending on the inputs.

### Match whole words

> Can be overridden on an entry level.

Entries with keys containing only one word will be matched only if the entire word is present in the search text. Enabled by default.

For example, if the setting is enabled and the entry key is "king", then text such as "long live the king" would be matched, but "it's not to my liking" wouldn't.

**Important:** this setting can have a detrimental effect when used with languages that don't use whitespace to separate words (e.g. Japanese or Chinese). If you write entries in these languages, it is advised to keep it off.

### Alert on overflow

Shows an alert if the activated World Info exceeds the allocated token budget.



### World Info Entry

#### Entry Title / Memo

A text field for your convenience to label your entries, which is not utilized by the AI or any of the trigger logics.

If empty, can be backfilled using the entries' first key by clicking on the "Fill empty memos" button.


#### Key

A list of keywords that trigger the activation of a World Info entry. Keys are not case-sensitive by default (this is [configurable](#case-sensitive-keys)).

##### Key Input

There are two modes to enter keywords, each with a slightly different UI. In ⌨️ *plaintext mode* (default), keys can be entered as a comma-separated list in a single text field. Regexes can be included too, but they don't have any special highlighting. In ✨ *fancy mode*, the keys appear as separate elements and regexes will be highlighted as such. The control supports editing and deleting keys. The mode can be switched via the inline button inside the input control.


#### Use Group Scoring

When this setting is enabled globally or per entry, the number of activated entry keys determines the group winner selection. Only the subset of a group with the highest number of key matches will be left to be activated by Group Weight or Inclusion Priority - the rest will be deactivated and removed from the group.

Use this to give more specificity for individual entries in large groups. For example, they can have a common key and a specific key. A random entry will be inserted when a specific key is not provided, and vice versa.

The score calculation logic for primary keys is 1 match = 1 point.

For secondary keys, the interaction depends on the chosen Selective Logic:

1. AND ANY: 1 secondary match = 1 point.
2. AND ALL: 1 point for every secondary key if they all match.
3. NOT ANY and NOT ALL: no change.

Example:

* Entry 1. Keys: song, sing, Black Cat. Group: songs
* Entry 2. Keys: song, sing, Ghosts. Group: songs

The input `sing me a song` can activate either entry (both activated 2 keys), but `sing me a song about Ghosts` will activate only Entry 2 (activated 3 keys).

#### Automation ID

Allows to integrate World Info entries with [STscripts](/For_Contributors/st-script.md) from Quick Replies extension. If both the quick reply command and the WI entry have the same Automation ID, the command will be executed automatically when the entry with a matching ID is activated.

Automations are executed in the order they are triggered, adhering to your designated sorting strategy, combining the [Character Lore Insertion Strategy](#character-lore-insertion-strategy) with the 'Priority' sorting. Which leads to [Blue Circle](#strategy) entries processed first, followed by others in their specified 'Order'. Recursively triggered entries will be processed after in the same order.

The script command will run only once if multiple entries with the same Automation ID are activated.

#### Character Filter

A list of character names for which this entry can be activated. If this list is not empty, the entry will only be activated for characters whose names are on the list. When a tag is selected, the entry will only be activated for characters that have that specific tag.

"Exclude" mode inverts the filter, meaning that the entry will be activated for all characters except those that are added to the list or that have the selected tag(s).

#### Triggers

The generation types for which this World Info entry can be activated. If nothing is selected, the entry can be activated for all generation types. If one or more are selected, the entry will only be activated for those specific generation types:

* **Normal:** Regular message generation request.
* **Continue:** When the Continue button is pressed.
* **Impersonate:** When the Impersonate button is pressed.
* **Swipe:** When the generation is triggered by swiping.
* **Regenerate:** When the Regenerate button is pressed in solo chats.
* **Quiet:** Background generation requests, usually triggered by [extensions](/extensions/index.md) or [STscript](/For_Contributors/st-script.md) commands.

!!!
The "Regenerate" trigger is not available in group chats as it uses different regeneration logic: all messages from the last reply are deleted, and messages are queued using the "Normal" generation type according to the chosen [Group reply strategy](/Usage/Characters/groupchats.md#reply-order-strategies).
!!!
