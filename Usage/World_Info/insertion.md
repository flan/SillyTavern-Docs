---
order: 90
icon: tasklist
route: /usage/core-concepts/worldinfo/insertion/
templating: false
---

# Prompt Insertion

## Character Lore

Optionally, one World Info file could be assigned to a character to serve as a dedicated lore source across all chats with that character (including groups).

To do that, navigate to a Character Management panel and click a globe button, then pick World Info from a dropdown list and click "Ok".

To unbind or change character lore, Shift-click the globe button. If on mobile, click "More..." and then "Link World Info".



#### Entry Content

The text that is inserted into the prompt upon entry activation.

#### Insertion Order

Numeric value. Defines a priority of the entry if multiple were activated at once. Entries with larger order numbers will be inserted closer to the end of the context as they will have more impact on the output. For example, an entry with Order number 100 will appear in the context before an entry with Order number 250.

#### Insertion Position

* **Before Char Defs:** World Info entry is inserted before the character's description and scenario. Has a moderate impact on the conversation.
* **After Char Defs:** World Info entry is inserted after the character's description and scenario. Has a greater impact on the conversation.
* **Before Example Messages:** The World Info entry is parsed as an example dialogue block and inserted before the examples provided by the character card.
* **After Example Messages:** The World Info entry is parsed as an example dialogue block and inserted after the examples provided by the character card.
* **Top of AN:** World Info entry is inserted at the top of Author's Note content. Has a variable impact depending on the Author's Note position.
* **Bottom of AN:** World Info entry is inserted at the bottom of Author's Note content. Has a variable impact depending on the Author's Note position.
* **@ D:** World Info entry is inserted at a specific depth in the chat (Depth 0 being the bottom of the prompt).
  * ⚙️ - as a system role message
  * 👤 - as a user role message
  * 🤖 - as an assistant role message
* **Outlet:** World Info entry is not injected automatically. Instead, its content is stored under a named outlet so you can decide exactly where it appears in the prompt by calling it with the [`{{outlet::Name}}` macro](#outlet-name).

Example Message entries will be formatted according to the prompt-building settings: Instruct Mode or Chat Completion prompt manager. They also follow the Example Messages Behavior rules: being gradually pushed out on full context, always kept, or disabled altogether.

If your Author's Note is disabled (Insertion Frequency = 0), World Info entries in A/N positions will be ignored!

#### Outlet Name

When the **Outlet** insertion position is selected, an additional **Outlet Name** field becomes available for the entry. The name that you provide here groups entries together and defines the token that you will use to pull them into the prompt manually.

Use the `{{outlet::YourName}}` macro in the [Prompt Manager](../Prompts/prompt-manager.md) or [Advanced Formatting](../Prompts/advancedformatting.md) prompt fields. When the prompt is built, the macro is replaced with the combined content of every World Info entry that shares the same outlet name, separated by newlines, sorted by their [Insertion Order](#insertion-order) value.

If an outlet entry is missing a name it will be skipped during generation, so make sure to fill in the field. Outlet names support autocomplete based on the names you have already used to make it easy to reuse consistent labels.

##### Limitations and caveats

* Placing outlet macros inside World Info entries is not supported and will not work. This conflicts with the evaluation order of World Info and may lead to infinite loops.
* Nesting outlets is not supported. You cannot place an outlet macro inside another outlet's content. Same as above, this may lead to infinite loops.
* Character card fields (Description, Personality, Scenario, etc.) cannot expand outlets. Those fields are parsed early so they can act as [additional matching sources](./structure.md#additional-matching-sources) for World Info triggers, which means outlets are not available when their text is processed. Use another macro-aware field if you need to place outlet content in the prompt body instead.
* The Author's Note editor also cannot resolve outlets. To place outlet content around the Author's Note, assign the entries to **Top of AN** or **Bottom of AN** insertion positions instead of relying on the macro.
* Outlet names are case-sensitive. The `{{outlet::}}` macro must use exactly the same capitalization as the entry's **Outlet Name**, otherwise no content is returned.
* Leading or trailing spaces in an outlet name are ignored when you call the macro, so names saved with extra whitespace will not match. Avoid padding names so they can be resolved correctly.
* Outlet macros that have no content assigned to them will be replaced with an empty string.



#### Inclusion Group

Inclusion groups control how entries are selected when multiple entries with the same group label are triggered simultaneously. If multiple entries having the same group label were activated, only one will be inserted into the prompt.

By default, the chosen entry is selected randomly based on their Group Weight (default is 100 points) — the higher the number, the higher the probability of selection. This allows for a random selection among the triggered entries, adding an element of surprise and variety to interactions.

A single entry can be part of multiple inclusion groups if they are defined as a comma-separated list. The same logic as explained above will apply. If that entry is triggered, it will *disable* all other entries that are part of any of its groups. Therefore, if any of the groups are activated, this entry will not be activated.

#### Prioritize Inclusion

To provide more control over which entries are activated via [Inclusion Group](./structure.md#inclusion-group), you can use the 'Prioritize Inclusion' setting. This option allows you to specify deterministically which entry to choose instead of randomly rolling Group Weight chances.

If multiple entries having the same group label and this setting turned on were activated, the one with the highest 'Order' value will be selected. This is useful for creating fallback sequences via inclusion groups. For example to prioritize low-depth entries with more emphasis, or to choose a specific instruction on setting the scene over another if both are valid.


