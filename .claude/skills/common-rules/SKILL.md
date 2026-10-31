---
name: common-rules
description: Rules every worker follows.
---

## Scope

- Deliver what the story and `plan.md` ask, at the scope intended. If a better approach exists, say so in one sentence and still do what was asked, because unapproved extras widen every review.
- Add a dependency only when `plan.md` names it. Each one is code the team must update and trust.
- When the requirements or the API spec don't cover what you need, ask the team lead instead of guessing. A guess becomes a decision nobody approved.

## Code

- Read before you use: never assume a component's props, an API field or a file's contents without opening it.
- Write code that reads like the code around it: same naming, comment density and idioms. A reader should not be able to tell which lines an agent wrote. Where the code around it breaks your rule set, the rule set wins for the lines you write, and code the story does not touch stays as it is.
- Name things by what they are in the domain, using the words from the requirements and domain docs. Generic names like `data`, `item`, `handle` or `utils` force every reader to open the code to learn what it holds.
- Comment only what the code cannot say: a reason, a constraint, a workaround. Never narrate what the code does or what you changed; that belongs in the commit message.
- Handle errors only where they can happen: at user input and external calls. Guards for impossible cases hide real bugs and bury the logic.
- Build no helper, abstraction or option for a single use or an imagined future need. The right amount of structure is what the current story needs.
- Solve the problem in general. Never shape code to the test inputs or special-case a test to make it pass; if a test looks wrong, say so.
- Leave no debug output, commented-out code or mock data behind. They reach production because nobody owns them.
