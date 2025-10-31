

#### Strategy

1. 🔵 (Blue Circle) = The entry would always be present in the prompt.
2. 🟢 (Green Circle) = The entry will be triggered only in the presence of the keyword.
3. 🔗 (Chain Link) = The entry is allowed to be inserted by embedding similarity.

Each Entry also has a toggle that allows you to enable or disable the entry.

#### Probability (Trigger %)

This value acts like an additional filter that adds a chance for the entry NOT to be inserted when it is activated by any means (constant, primary key, recursion).

1. Probability = 100 means that the entry will be inserted on every activation.
2. Probability = 50 means that the entry will be inserted with a 1:1 chance.
3. Probability = 0 means that the entry will NOT be inserted (essentially disabling it).

Use this to create random events in your chats. For example, every message could have a 1% chance of waking up an Elder God if its name is mentioned in the message.



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

#### Key

A list of keywords that trigger the activation of a World Info entry. Keys are not case-sensitive by default (this is [configurable](#case-sensitive-keys)).

##### Regular Expression (Regex) as Keys

Keys allow a more flexible approach to matching by supporting regex. This makes it possible to match more dynamic content with optional words or characters, spacing, and all the other utilities that regex provides.  
If a defined key is a valid regex (Javascript regex style, with `/` as delimiters. All flags are allowed), it will be treated as such when checking whether an entry should be triggered. Multiple regexes can be entered as separate keys and will work alongside each other. Inside a regex, commas are possible. Plaintext keys do not support commas, as they are treated as key separators.  

An example of a use-case for advanced regex matching:  
An entry/instruction that should be inserted, when char is doing a weather-related action

```js
/(?:{{char}}|he|she) (?:is talking about|is noticing|is checking whether|observes) (?:the )?(rainy weather|heavy wind|it is going to rain|cloudy sky)/i
```

For more information on Regex syntax and possibilities: [Regular expressions - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_expressions)

###### Advanced Regex Per-Message Matching

ST prefixes every chat message in the WI scan buffer with `character name:` and after v1.12.6, concatenates prepends them using the character value 1 (`\x01`).  
This means you can match specific input or output from a certain character using a regex tied to that separation character.

For example, to match only the user saying "hello", you could use the following regex:

```js
/\x01{{user}}:[^\x01]*?hello/
```

##### Key Input

There are two modes to enter keywords, each with a slightly different UI. In ⌨️ *plaintext mode* (default), keys can be entered as a comma-separated list in a single text field. Regexes can be included too, but they don't have any special highlighting. In ✨ *fancy mode*, the keys appear as separate elements and regexes will be highlighted as such. The control supports editing and deleting keys. The mode can be switched via the inline button inside the input control.

#### Optional Filter

Comma-separated list of additional keywords in conjunction with the primary key.
If no arguments are provided, this flag is ignored.
Supports logic for AND ANY, NOT ANY, or NOT ALL

1. AND ANY = Activates the entry only if the primary key and Any one of the optional filter keys are in scanned context.
2. AND ALL = Activates the entry only if the primary key and ALL of the optional filter keys are present.
3. NOT ANY = Activates the entry only if the primary key and None of the optional filter keys are in scanned context.
4. NOT ALL = Prevents activation of the entry despite primary key trigger, if all of the optional filters are in scanned context.

These keys also support [regex](#regular-expression-regex-as-keys).


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

#### Additional matching sources

By default World Info Entries are matched only against content from the current conversation. These options allow you to match the entry against different character information that does not show up in the chat, or even persona information. This is useful when you want to have a wide range of entries that are to be used between several characters but don't want to have to manage large lists of tags, or don't want to have to update character filter lists every time you create a new one. This also allows you to match entries based on the persona you have active.

* **Character Description**: Matches against the character description.
* **Character Personality**: Matches against the character personality summary, found under Advanced Definitions.
* **Scenario**: Matches against the character specified scenario, found under Advanced Definitions.
* **Persona Description**: Matches against the current selected persona's description.
* **Character's Note**: Matches against the character's note, which can be found under Advanced Definitions.
* **Creator's Notes**: Matches against the character creator's notes, which can be found under Advanced Definitions. The creator's notes are usually not included in the prompt.
