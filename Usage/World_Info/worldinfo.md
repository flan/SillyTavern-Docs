---
order: 100
icon: pencil
route: /usage/core-concepts/worldinfo/design/
templating: false
---

# World Info design

At the most basic level, entries in a World Info Lorebook are like pages in an encyclopedia: there are articles and there are a few keywords that can be used to find them.

Within SillyTavern, there is some additional metadata, like [how to interpret those keywords and how entries are ranked against each other](#anatomy-of-a-lorebook-entry).

There is also a specific process by which SillyTavern [assembles World Info into a request for the LLM](#prompt-insertion).

## Anatomy of a Lorebook entry

Every entry has a large number of fields, presented here in no particular order.

*Note: the order in which Lorebook entries appear is entirely for your benefit as the author: group them by purpose, sort them alphabetically... Do whatever makes sense to you.*

*Note: In the first row of every entry, there is an unlabeled "enable" toggle that lets the entry be quickly turned off entirely. This is useful for both testing and swapping attributes of a character.*

### Title/Memo
Descriptive text to help the reader know what this entry is for. It is *not* sent to the LLM.

Empty titles can be auto-populated with the "Fill empty memos" button at the top of the Lorebook list.

### Strategy
- 🔵 (Blue Circle) = This entry will always be activated.
- 🟢 (Green Circle) = This entry will be activated only when its keywords are matched.
- 🔗 (Chain Link) = This entry will be considered based on [Vector Storage Matching](./advanced.md#vector-storage-matching).
  - The idea here is to let the LLM score the entry according to its own understanding of language, then compare those scores to the context being evaluated to determine whether they are sufficiently similar to warrant inclusion.
  - This is a common cause of confusion when writing Lorebooks and it is not recommended while you are still learning how to interact with your LLM model, SillyTavern, and story-generation in general.

*Note: activating an entry is not sufficient for it to be [included in the final prompt](#prompt-insertion): there is a [limited amount of space available](./global.md#context--and-budget-cap) and some entries may be dropped.*

### Position, Depth, and Order

A means of ordering where the entry will appear in the prompt sent to the LLM; see [notes on Position in the assembly section](#position).

### Trigger %
The probability that the entry will be activated when all selection criteria are met, used to introduce some randomness. `100` means "always" and `0` means "never".

Example uses of this could be to change the weather or to have a 1% chance of waking an Elder God with each message if your story needs a looming high-stakes threat.

### Keywords

These are the main matching rules for Lorebook Entries.

#### Primary Keywords
A comma-delimited list of strings (text) that are used to match this entry for activation.

Example: `cat,kitten, feline` (whitespace is trimmed).

#### Optional Filter
A comma-delimited list of strings (text) that are used to further match this entry for activation.

Example: `dog, puppy,canine` (whitespace is trimmed).

#### Logic
When an Optional Filter is specified, this will determine how to interpret it using common Boolean logic:
  - `AND ANY` activates the entry only if one of the Primary Keywords and *any* of the Optional Filter keywords are present.
  - `AND ALL`: activates the entry only if one of the Primary Keywords and *all* of the Optional Filter keywords are present.
  - `NOT ANY`: activates the entry only if one of the Primary Keywords and *none* of the Optional Filter keywords are present.
  - `NOT ALL`: activates the entry only if one of the Primary Keywords and *all* of the Optional Filter keywords are present.
    - If the Optional Filter is `dog, puppy, canine` and both `dog` and `puppy` are present, but `canine` is absent, the entry will be activated.

#### Input modes
There are two modes available to enter keywords, each with a slightly different UI.

In ⌨️ *plaintext mode* (default), keywords are entered as a comma-separated list in a single text field, as illustrated above; [regular expressions](./advanced.md#regular-expressions-regex-as-keys) can be specified, too. In ✨ *fancy mode*, keys will instead show up as separate items and regular expressions will be given special highlighting.

It is possible to add, edit, and delete all types of keys in either mode; the input mode may be selected by clicking on the icon in the text-field.

### Scan Depth

> Can inherit from [Global Activation Settings](./global.md).

Defines how many messages in the chat history should be scanned for World Info keys, starting from the most recent.

* When set to `0`, only recursed entries and Author's Note are evaluated.
* When set to `1`, SillyTavern only scans the last message.
* When set to `2`, the two most recent messages are considered; there is no practical upper limit.

### Case-Sensitive

> Can inherit from [Global Activation Settings](./global.md).

This makes [Keyword](#keywords) matching more strict, requiring that words match exactly in terms of capitalization.

This is mostly useful if your keys are proper nouns within your chat, like the names of important people or cities.

It is likely quite unhelpful if your chat takes the form of text-message exchanges where the style is often intentionally lazy.

For example, when this setting is active, `rose` will *not* match `Rose` because `'r' != 'R'`.

### Whole Words

> Can inherit from [Global Activation Settings](./global.md).

This makes [Keyword](#keywords) matching more strict, requiring that each keyword matches a word in the input based on common word-boundary markers (spaces, periods, hyphens).

This is usually a good idea because `cat` probably shouldn't match `concatenate`.

If you want to match words that start or end with a certain sequence of characters, consider [regular expressions](./advanced.md#regular-expressions-regex-as-keys).

*Important: this setting is often incompatible with languages that don't use whitespace to separate words (e.g. Japanese, Chinese). If you write keywords in these languages, it is advised to turn it off.*

### Group Scoring
<sub>*Though visually displaced in the UI, this is connected to [Inclusion Groups](#inclusion-group).*</sub>

> Can inherit from [Global Activation Settings](./global.md).

When enabled, this adds a pre-selection pass to activation candidacy: each entry that would be activated is scored based on how many of its keywords match and only those with the strongest results are considered (typically just one, unless there is a tie). The use-case for this is when members of a group have a common set of [Primary Keywords](#primary-keywords) and a wide range of [Optional Filter keywords](#optional-filter).

The scoring formula is as follows:
- For each Primary Keyword that matches, 1 point is awarded
- When using an Optional Filter:
  - `AND ANY`: for each keyword that matches, 1 point is awarded
  - `AND ALL`: if all keywords match, 1 point is awarded per defined keyword
  - `NOT ANY` and `NOT ALL`: no additional points are awarded

Example:

* Entry 1 (Inclusion Group: "songs")
  * Keys: {`song`, `sing`, `Black Cat`}
* Entry 2 (Inclusion Group: "songs")
  * Keys: {`song`, `sing`, `Ghosts`}

The input `sing me a song` can activate either entry (in this case, 2 keys match: `song` and `sing`), but `sing me a song about Ghosts` will activate only Entry 2 (3 points for Entry 2 versus only 2 points for Entry 1).

### Automation ID
This allows you to specify an arbitrary ID as a number; if you do so *and* you create a Quick Reply function with the same ID (using the Quick Replies [extension](/extensions/index.md)), then whenever this entry is activated, that function will also trigger, allowing you to run [STscripts](/For_Contributors/st-script.md) contextually.

Triggered functions will be queued and run in the sequence that they would be [inserted into your prompt](#prompt-insertion), [Blue Circle](#strategy) entries first, then others according to [Order](#order). Recursively activated entries will be processed afterwards, using the same ordering mechanism.

If multiple entries would trigger the same Quick Reply function (based on ID-matching), that function will only be triggered once.

### Content
The only part of the entry that the LLM will actually see. When activated, this text, after being processed to evaluate macros like `{{char}}` and `{{random}}`, will be prepared for inclusion in the prompt body.

#### Recursion
When an entry [qualifies for inclusion in the prompt](#prompt-insertion), its content will then be used in another activation check, potentially activating even more entries. Each cycle of this process is a recursive layer.

The following flags control how recursion works with this entry, in addition to the [Global Activation Settings](./global.md):

- **Non-recursable**
  - This entry *cannot* be activated through recursion.
- **Delay until recursion**
  - This entry can *only* be activated through recursion.
- **Prevent further recursion**
  - This entry cannot be used to activate *other* entries through recursion

#### Budget
Each activated entry becomes a candidate for inclusion in the prompt that will be sent to the LLM. However, because there is a limited amount of context-space available in any given request, if there are are more prompts than can fit, some will be discarded.

Setting the **Ignore budget** flag will ensure that this entry will not be discarded, exempting it from budgetary considerations. This may negatively affect the amount of space available to prepare a response, however, so be very careful and apply it only to small or absolutely essential entries.

### Inclusion Group
An Inclusion Group is a collection of entries with this string as a shared label. When set, only one member of that group may be selected at any given time, selected from among those that activated.

This may be useful for cases where a character might have one of a selection of demi-human traits, but they would be in conflict with each other if both activated at the same time, such as "cat ears" and "dog ears".

A single entry may be made part of multiple inclusion groups by providing a comma-separated list, like `cat, dog,cow`. When such an entry is selected, all other entries in all associated groups will be disabled, preventing their own selection. Likewise, if a member of one of those groups is selected, then all other entries belonging to that group, including the entry with multiple memberships, is disabled: it does not get a chance to represent another group in the current prompt.

The **Prioritize** flag may be used to indicate that this entry is dominant, with ties being settled based on [Order](#order). This supersedes [Group Weight](#group-weight) when there might otherwise be competition and may allow generally low-scoring entries a chance to be selected more often.

### Group Weight
If multiple members of a group are set to be activated, this influences the dice-roll to determine which will be chosen: `probability = weight / sum(weights[entry_1], weights[entry_2], ...)`

### Sticky
> See [Timed Effects](./advanced.md#timed-effects) for more information.

This allows an entry to remain active for an arbitrary number of messages after it was activated, to help it have a lasting effect on the story.

Useful when randomly waking Elder Gods or learning about a character's traumatic past.

### Cooldown
> See [Timed Effects](./advanced.md#timed-effects) for more information.

This allows an entry to only be activated on occasion.

You *just* finished dealing with a surprise phone-call; you don't want to do that again any time soon.

### Delay
> See [Timed Effects](./advanced.md#timed-effects) for more information.

This ensures that an entry will not be triggered before a certain number of messages have been exchanged in the chat.

This helps to ensure your initial "where am I?" question is not met with a detailed breakdown of a millennia of political machinations.

### Filter to Characters or Tags
When populated, only the listed characters, or characters with any of the selected tags, will be able to activate this entry.

Setting the **Exclude** flag inverts this behavior, meaning that the indicated characters and tags *cannot* activate this entry.

### Filter to Generation Triggers
Limits the events that can trigger activation of this entry. When none are explicitly specified, all methods are enabled.

- **Normal**: evaluate this entry whenever the user provides a message.
- **Continue**: evaluate this entry whenever the "Continue" button is pressed.
- **Impersonate**: evaluate this entry whenever the "Impersonate" button is pressed.
- **Swipe**: evaluate this entry whenever swiping occurs.
- **Regenerate**: evaluate this entry whenever the Regenerate button is pressed in solo chats.
- **Quiet**: evaluate this entry as part of background tasks, such as actions taken by [extensions](/extensions/index.md) or [STscript](/For_Contributors/st-script.md) commands.

*Note: the **Regenerate** trigger is not available in group chats because they use different regeneration logic: all messages following the last reply are deleted, then messages are queued using the **Normal** generation-type according to the chosen [Group reply strategy](/Usage/Characters/groupchats.md#reply-order-strategies).*

### Additional Matching Sources

By default, entries are only evaluated against the scan-buffer specified by [Scan Depth](#scan-depth).

These options allow other sources of text to be considered, which may be useful when you want to include information that is not part of the chat-history itself or may even be descriptive metadata.

- **Character Description**
  - Includes `{{char}}`'s **Description** text.
- **Character Personality**
  - Includes `{{char}}`'s **Personality** text, located under Advanced Definitions.
- **Scenario**
  - Includes `{{char}}`'s **Scenario** text, located under Advanced Definitions.
- **Persona Description**
  - Includes `{{user}}`'s **Description** text.
- **Character's Note**
  - Includes `{{char}}`'s **Character Note** text, located under Advanced Definitions.
- **Creator's Notes**
  - Includes `{{char}}`'s **Creator's Notes** text, located under Advanced Definitions; this is typically entirely commentary.

## Prompt Insertion

Once all applicable entries have been activated, it becomes necessary to assemble the prompt to be sent to the LLM.

For this process, there are a few considerations:

1. There is a [finite context budget](./global.md#context--and-budget-cap).
1. There are multiple [Insertion Positions](#position) that any given entry could occupy, affecting the order in which the LLM processes it and its proximity to other data for context.
1. Every entry has metadata, like [Order](#order), that influence how favorably in their respective positions they will appear, with lower-priority entries being discarded as needed. 

### Position

This somewhat complicated option is worth taking the time to learn, but the default setting will generally work well enough.

- **Before Character Definitions** (￪Char): the World Info entry is inserted before the character's description and scenario.
 - This has a moderate impact on the chat and might not help much if it refers to information about the character that has not yet been introduced.
- **After Character Definitions** (￬Char): the World Info entry is inserted after the character's description and scenario.
  - This has a somewhat significant impact on the chat, allowing additional information about the character to be presented while in close proximity to the character's core definition.
- **Before Example Messages** (￪EM): the World Info entry is parsed as an example dialogue block and inserted before any examples provided by the character card.
- **After Example Messages** (￬EM): the World Info entry is parsed as an example dialogue block and inserted after any examples provided by the character card.
- **Top of Author's Note** (￪AN): World Info entry is inserted above Author's Note content.
  - The impact varies with the position of the Author's Note, which is configurable.
- **Bottom of Author's Note** (￬AN): World Info entry is inserted below Author's Note content.
  - The impact varies with the position of the Author's Note, which is configurable.
- **Depth relative to the end of a prompt-block** (@D):
  - ⚙️ (gear): system-role messages
  - 👤 (person): user-role messages
  - 🤖 (robot): assistant-role messages
- **Outlet**: The World Info entry is not inserted automatically. Instead, its content is stored under a named Outlet so you can decide exactly where it appears in the prompt; see [Outlet Name](#outlet-name) for details.

Example Message entries will be formatted according to the [prompt-building settings](/Usage/Prompts/index.md): either Instruct Mode or the Chat Completion prompt manager. They also follow the [Example Messages Behavior rules](/Usage/User_Settings/index.md#chatmessage-handling): being gradually pushed out as context fills, marked as always kept, or disabled altogether.

If your Author's Note is disabled (Insertion Frequency = `0`), World Info entries set to insert relative to the Author's Note will be discarded.

#### Depth
When one of the "@D" options is chosen, this sets the relative position of the entry, with `0` being at the bottom of the prompt-block, `1` being one entry prior, `2` being two entries prior, and so forth.

Messages closer to the end of the prompt generally have more influence on the response.

#### Order
Order defines the priority of an entry, used to serve as a tiebreaker when many are activated simultaneously.

Entries with larger Order numbers will be inserted closer to the end of the context, which gives them more impact on the response. For example, an entry with an Order of `100` will appear in the context before an entry with an Order of `250`.

#### Outlet Name

When an entry is assigned the "Outlet" insertion position, an additional Outlet Name field appears within the SillyTavern UI. The name entered within this field becomes the key for accessing a dynamic macro: if the given name is `banana`, then the macro `{{outlet::banana}}` will cause the corresponding World Info entry or entries to be produced in its place.

Multiple Outlets may share the same name, which will cause the accumulated World Info entries to be appended to each other, separated by newlines (`\n`), sorted by their [Order](#order) value.

Outlets created in this way are intended for use with the [Prompt Manager](../Prompts/prompt-manager.md) and [Advanced Formatting](../Prompts/advancedformatting.md) prompt fields.

*Note: An Outlet lacking a name will be skipped, so be sure not to overlook the field. The UI element supports autocomplete to make it easier to ensure that Outlets that should share names can be found quickly and have identical spelling.*

##### Limitations and caveats

* Placing Outlet macros inside World Info entries is not supported and will not work. This conflicts with the evaluation order of World Info and may lead to infinite loops.
* Nesting Outlets is not supported. You cannot place an Outlet macro inside another Outlet's content. As with nesting inside of World Info entries, this may lead to infinite loops.
* Character card fields (Description, Personality, Scenario, etc.) cannot expand Outlets. Those fields are parsed early so they can act as [additional matching sources](./worldinfo.md#additional-matching-sources) for World Info activation, which means Outlets are not available when their text is processed. Use another macro-aware field if you need to place Outlet content in the prompt body instead.
* The Author's Note editor also cannot resolve Outlets. To place what would be Outlet content around the Author's Note, assign the entries to the **Top of Author's Note** or **Bottom of Author's Note** insertion positions instead of invoking the macro.
* Outlet names are case-sensitive. The `{{outlet::}}` macro must use exactly the same capitalization as the entry's [Outlet Name](#outlet-name), otherwise no content will be returned.
* Leading or trailing spaces in an outlet name are ignored when you call the macro, so names saved with extra whitespace will not match. Avoid padding names so they can be resolved correctly.
* Outlet macros that have no content assigned to them will be replaced with an empty string.
