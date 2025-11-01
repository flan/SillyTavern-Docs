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

Empty titles can be auto-populated with the "Fill empty memos" button at the top of the Lobrebook list.

#### Strategy
- 🔵 (Blue Circle) = This entry will always be activated.
- 🟢 (Green Circle) = This entry will be activated only when its keywords are matched.
- 🔗 (Chain Link) = This entry will be matched based on [special magic math](./advanced.md#vector-storage-matching).
  - The idea here is to let the LLM score the entry according to its own understanding of language, then compare those scores to the context being evaluated to determine whether they are sufficiently similar to warrant inclusion.
  - This is a common cause of confusion when writing Lorebooks and it is not recommended while you are still learning how to interact with your LLM model, SillyTavern, and story-generation in general.

*Note: activating an entry is not sufficient for it to be [included in the final prompt](./insertion.md): there is a [limited amount of space available](./global.md#context--budget) and some entries may be dropped.*

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

##### Input modes
There are two modes available to enter keywords, each with a slightly different UI. In ⌨️ *plaintext mode* (default), keys are entered as a comma-separated list in a single text field, as illustrated above. [Regular expressions](#regular-expressions-regex-as-keys) can be specified, too. In ✨ *fancy mode*, keys will instead show up as separate items and regular expressions will be given special highlighting. It is possible to add, edit, and delete all types of keys in either mode; the input mode may be selected by clicking on the icon in the text-field.

#### Scan Depth

> Can inherit from [Global Activation Settings](./global.md).

Defines how many messages in the chat history should be scanned for World Info keys, starting from the most recent.

* When set to 0, only recursed entries and Author's Note are evaluated.
* When set to 1, SillyTavern only scans the last message.
* When set to 2, the two most recent messages are considered; there is no practical upper limit.

#### Case-Sensitive

> Can inherit from [Global Activation Settings](./global.md).

This makes [Keyword](#keywords) matching more strict, requiring that words match exactly in terms of capitalisation.

This is mostly useful if your keys are proper nouns within your chat, like the names of important people or cities. It is likely quite unhelpful if your chat takes the form of text-message exchanges.

For example, when this setting is active, `rose` will *not* match `Rose` becuse `'r' != 'R'`.

#### Whole Words

> Can inherit from [Global Activation Settings](./global.md).

This makes [Keyword](#keywords) matching more strict, requiring that each keyword matches a word in the input based on common word-boundary markers (spaces, periods, hyphens).

This is usually a good idea because `cat` likely shouldn't match `concatenate`.

If you want to match words that start or end with a certain sequence of characters, consider [regular expressions](#regular-expressions-regex-as-keys).

*Important: this setting is often incompatible with languages that don't use whitespace to separate words (e.g. Japanese, Chinese). If you write entries in these languages, it is advised to turn it off.*

#### Group Scoring
<sub>*Though visually displaced in the UI, this is connected to [Inclusion Groups](#inclusion-group).*</sub>

> Can inherit from [Global Activation Settings](./global.md).

When enabled, this adds a pre-selection pass to activation candidacy: each entry that would be activated is scored based on how many of its keywords match and only those with the strongest results are considered. The use-case for this is when members of a group have a common set of [Primary Keywords](#primary-keywords) and a wide range of [Optional Filter keywords](#optional-filter). A random entry will be inserted when a specific key is not provided and vice versa.

The scoring formula is as follows:
- For each Primary Keyword that matches, 1 point is awarded
- When using an Optional Filter:
  - AND ANY: for each keyword that matches, 1 point is awarded
  - AND ALL: if all keywords match, 1 point is awarded per defined keyword
  - NOT ANY and NOT ALL: no additional points are awarded

Example:

* Entry 1 (Inclusion Group: "songs")
  * Keys: {`song`, `sing`, `Black Cat`}
* Entry 2 (Inclusion Group: "songs")
  * Keys: {`song`, `sing`, `Ghosts`}

The input `sing me a song` can activate either entry (in this case, 2 keys match: `song` and `sing`), but `sing me a song about Ghosts` will activate only Entry 2 (3 points for Entry 2 versus only 2 points for Entry 1).

#### Automation ID
This allows you to specify an arbitrary ID; if you do so *and* you create a Quick Reply function with the same ID (using the Quick Replies [extension](/extensions/index.md)), then whenever this entry is activated, that function will also trigger, allowing you to run [STscripts](/For_Contributors/st-script.md) contextually.

Triggered functions will be queued and run in the order that they would be [inserted into your prompt](./insertion.md), [Blue Circle](#strategy) entries first, then others according to [Order](#order). Recursively activated entries will be processed afterwards, using the same ordering mechanism.

If multiple entries would trigger the same Quick Reply function (based on ID-matching), that function will only be triggered once.

#### Content
The only part of the entry that the LLM will actually see. When activated, this text, after being processed to evaluate macros like `{{char}}` and `{{random}}`, will be prepared for inclusion in the prompt body.

##### Recursion
When an entry [qualifies for inclusion in the prompt](./insertion.md), its content will then be used in another activation check, potentially activating even more entries. Each cycle of this process is a recursive layer.

The following flags control how recursion works with this entry, in addition to the [Global Activation Settings](./global.md):

- Non-recursable
  - This entry *cannot* be activated through recursion.
- Delay until recursion
  - This entry can *only* be activated through recursion.
- Prevent further recursion
  - This entry cannot be used to activate *other* entries through recursion

##### Budget
Each activated entry becomes a candidate for inclusion in the prompt that will be sent to the LLM. However, because there is a limited amount of context-space available in any given request, if there are are more prompts than can fit, some will be discarded.

Setting the "Ignore budget" flag will ensure that this entry will not be discarded, exempting it from budgetary considerations. This may negatively affect the amount of space available to prepare a response, however, so be very careful and apply it only to small or absolutely essential entries.

#### Inclusion Group
An Inclusion Group is a collection of entries with this string as a shared label. When set, only one member of that group may be triggered at any given time, selected from among those that activated.

This may be useful for cases where a character might have one of a selection of demi-human traits, but they would be in conflict with each other if both activated at the same time, such as "cat ears" and "dog ears".

The "Prioritize" flag may be used to indicate this this entry is dominant, with ties being settled based on [Order](#order).

#### Group Weight
If multiple members of a group are set to be activated, this influences the dice-roll to determine which will be chosen: `weight / sum(weights[first_entry], weights[second_entry], ...)`

#### Sticky
> See [Timed Effects](./advanced.md#timed-effects) for more information.

This allows an entry to remain active for an arbitrary number of messages after it was activated, to help it have a lasting effect on the story. Useful when randomly waking Elder Gods or learning about a character's traumatic past.

#### Cooldown
> See [Timed Effects](./advanced.md#timed-effects) for more information.

This allows an entry to stay dormant after being triggered for an arbitrary number of messages, to create some unpredictability. You might have deduced that you are being stalked by a terrible monster, but it won't reveal itself to you *just* yet.

#### Delay
> See [Timed Effects](./advanced.md#timed-effects) for more information.

This ensures that an entry will not be triggered before a certain number of messages have been exchanged in the chat. This helps to ensure your initial "where am I?" question is not met with a detailed breakdown of a millennia of political machinations.

#### Filter to Characters or Tags
When populated, only the listed characters, or characters with any of the selected tags, will be able to activate this entry.

Setting the "Exclude" flag inverts this behaviour, meaning that the indicated characters and tags *cannot* activate this entry.

#### Filter to Generation Triggers
Limits the events that can trigger activation of this entry. When none are explicitly specified, all methods are enabled.

- Normal: evaluate this entry whenever the user provides a message.
- Continue: evaluate this entry whenever the "Continue" button is pressed.
- Impersonate: evaluate this entry whenever the "Impersonate" button is pressed.
- Swipe: evaluate this entry whenever swiping occurs.
- Regenerate: evaluate this entry whenever the Regenerate button is pressed in solo chats.
- Quiet: evaluate this entry as part of background tasks, such as actions taken by [extensions](/extensions/index.md) or [STscript](/For_Contributors/st-script.md) commands.

*Note: the "Regenerate" trigger is not available in group chats because they use different regeneration logic: all messages following the last reply are deleted, then messages are queued using the "Normal" generation-type according to the chosen [Group reply strategy](/Usage/Characters/groupchats.md#reply-order-strategies).*

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

The text considered for World Info matching, the scan-buffer, is prefixed with `{{char}}:` or `{{user}}:` before every message when the [Include Names](./global.md#include-names) option is set. Starting with SillyTavern version v1.12.6, the scan-buffer also delimits messages with the non-printable character `\x01`. These two properties together allow a regular expression to look for text produced by a specific character, which can help to gate information in a Lorebook based on a sort of ownership.

For example, the following regex would trigger only when the user says "banana bread" somewhere in their message:

```javascript
/\x01{{user}}:[^\x01]*?.*?\bbanana bread\b.*?[^\x01]*?/i
```

(The `[^\x01]` parts avoid considering more than one message)
