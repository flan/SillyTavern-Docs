---
order: 80
icon: globe
route: /usage/core-concepts/worldinfo/global/
templating: false
---

# Global Activation Settings

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

### Context % (Budget)

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


