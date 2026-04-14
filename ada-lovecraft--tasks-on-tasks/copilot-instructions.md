## tasks-on-tasks

> when working on a todo list

# When working on Todo List

## Use notifications
to use notifications run the command `osascript -e display notification "{message}" with title "{title}"`. 
notification titles should follow this format: `<todo list/item name> [x/y]` where `x` is the item or section index you are displaying the notifaction for and `y` is the total items or sections included
notification messages should follow this format `<ACTION> :: <todo list/item name>`



## When Working on a Todo List

- notify the user you're begginning on a new list
- when beginning work on checklist items in a new subsection,
    - update the subsection's title in the todo doc  with `[IN PROGRESS]`
    - create an empty `Steps Taken` section in this subsection to be filled in as items are completed
- upon completing all check list items in a sub section,
    - update the subsection's title to end with `[COMPLETE]`
    - notify the user that you've completed the todo list
- upon completing all check list items in the dev story, notify the user

## When Working on a Todo Checklist Item

- Work on one check list item at a time
- notify the user you've started work on the item
- when you have completed the checklist item
    - append bullet items to the `Steps Taken` area that describe the changes you made to complete the checklist item
    - notify the user you've completed the item
    - update the todo document to mark the item as complete

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ada-lovecraft)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ada-lovecraft)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
