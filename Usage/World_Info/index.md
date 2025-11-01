---
order: 130
icon: globe
route: /usage/core-concepts/worldinfo/
templating: false
---

# World Info

**World Info is a mechanism available in SillyTavern to dynamically insert information into the prompts sent alongside your chat to help guide AI replies with supplementary information.**

The most common expression of World Info (often shortened to "WI") is that of a Lorebook, an indexed dictionary of information. The material within a Lorebook can range from past events from a character's life, to descriptions of cities or species, or even chemistry notes for a study assistant.

WI is typically loaded in a two-step process:
1. [Activation](./worldinfo.md#strategy) of pre-constructed Lorebooks.
   - During activation, the latest message, chat history, and other sources, such as Lorebooks, are considered to determine which WI entries are candidates for inclusion.
   - The activation process makes use of [Global Activation Settings](./global.md).
1. [Prompt Insertion](./worldinfo.md#prompt-insertion) of activated entries.
   - Once enumerated, relevant WI is sorted and inserted into the prompt to be sent to the AI according to a number of configurable prioritization strategies.

*When considering World Info, it is important to always be aware that AIs are statistical evaluation models. Simply making information available to them does not guarantee that they will act upon it or include it in any meaningful way in the output. The effectiveness of your Lorebooks depends strongly upon your ability to write useful entries, the nature and tuning of the AI model in use, and the relevance of the information to the response that should be generated.*

## Implementation

SillyTavern supports multiple concurrent Lorebooks in a chat:

* Any number of Active World Lorebooks, which apply to all chats.
* Up to one chat Lorebook, which can keep track of chat-specific history or customizations.
* Any number of character-bound Lorebooks, which apply to the character in all chats.
* Up to one persona-bound Lorebook, which applies to the persona in all chats.

The behavior of these Lorebooks are all functionally identical and entries activated from any will be given equal consideration for prompt-insertion.

### Association

World Lorebooks may be bound from the World Info screen, accessible from the main SillyTavern menu.

Chat Lorebooks may be bound by selecting a character or group chat, then clicking the `Chat Lore` button.

Character-bound Lorebooks may be bound by selecting a character, then clicking the `Character Lore` button.

Persona Lorebooks may be bound by selecting a Persona, then clicking the `Persona Lore` button.

## Character Lore

Optionally, one World Info file could be assigned to a character to serve as a dedicated lore source across all chats with that character (including groups).

To do that, navigate to a Character Management panel and click a globe button, then pick World Info from a dropdown list and click "Ok".

To unbind or change character lore, Shift-click the globe button. If on mobile, click "More..." and then "Link World Info".

## Suggestions and common pitfalls

* World Info is extremely flexible, limited only by the bounds of what can be expressed in text.
  * Despite the term "Lorebook", the contents do not need to be lore alone: they can be specific details like where the keys to a safe are kept, a list of flash card information, or event hints using ST Macros like `{{random}}` to add unpredictability.
* Activation keywords, titles, and other descriptive information outside of the `Content` field is never sent to the AI as part of the prompt body, so be sure that each World Info entry is self-contained.
  * An entry titled "Banana Bread" with `Content` of "This is tasty." will only be presented as `This is tasty.`, which will not help meaningfully guide the response.
* Activations can trigger more activations, allowing Lorebooks to chain related information through branching paths.
  * This allows a common detail, like the description of a species, to be referenced from multiple entries; see [notes on recursion](./global.md#recursive-scanning).
* Just as there is a limit to the context-window an AI can work with, there is a (configurable) limit to how much World Info SillyTavern will include in its prompts.
  * Activating everything and setting strategies to insert every entry will not only overload the AI with irrelevant information, it will also push out the parts that you actually want for the response.
  * Be judicious in selecting what gets activated and write short, concise entries that hint at what the AI should know, rather than verbose instructions that tell it what to do.

## Further reading

* [World Info Encyclopedia](https://rentry.co/world-info-encyclopedia): an exhaustive, in-depth guide to World Info and Lorebooks.
  - Authors: kingbri, Alicat, Trappu
