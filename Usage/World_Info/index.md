---
order: 130
icon: globe
route: /usage/core-concepts/worldinfo/
templating: false
---

# World Info

**World Info (also known as Lorebooks or Memory Books) is a powerful tool available in ST to insert prompts dynamically into your chat to help guide the AI replies.**

Commonly, World Info (WI for short) is used to  enhance the AI's understanding of the details in your fictional world, however you could use a World Info entry to insert ANYTHING that you would like to insert into the prompt.

It functions like a dynamic dictionary that only inserts relevant information from World Info entries when keywords associated with the entries are present in the message text.

The SillyTavern engine activates and seamlessly integrates the appropriate lore into the prompt, providing background information to the AI.

*It is important to note that while World Info helps guide the AI toward the desired content, it does not guarantee its appearance in the generated output messages. That depends on how good your model is at making use of additional information!*

## Pro Tips

* The World Info engine is a very powerful prompt management tool. Don't fixate on adding character lore alone, feel free to experiment.
* Activation keywords, titles, and other information that is not in the **Content** field is not inserted into context, so each World Info entry should have a comprehensive, standalone description.
* To create rich and detailed world lore, entries can be interlinked and reference one another by using recursive activation. See more on [Recursion](#recursive-scanning) below.
* SillyTavern offers flexible context budgeting for inserted background information. To conserve prompt tokens, it is advisable to keep entry contents concise.

## Further reading

* [World Info Encyclopedia](https://rentry.co/world-info-encyclopedia): Exhaustive in-depth guide to World Info and Lorebooks. By kingbri, Alicat, Trappu.

#### Entry Title / Memo

A text field for your convenience to label your entries, which is not utilized by the AI or any of the trigger logics.

If empty, can be backfilled using the entries' first key by clicking on the "Fill empty memos" button.
