# PROMPTS.md: Living Prompt Pack

> Module 3 · Prompt Chaining. Re-architect the build with prompt chains; capture the reusable ones here.

## How to use this pack

_Each prompt is a reusable step. Chain them: the output of one becomes the input to the next._

## Prompt chain: Prototype Improvement

### Step 1: Expand, build new screens in a strict sequence
```
Build the next phase of this app in a strict sequence:
1. Add a screen of the email of the CRM. Match the layout and style of the screen we already have Daily Action Queue.
2. Navigation: write the logic so to navigate from when the user clicks on Email, to actually be able to email the contact he needs to.


Build these in order so the Daily Action Queue screen is the anchor.
```

### Step 2: Behavior, hard-code the states
```
Apply the following logic constraints to the main flow:
- On fetch failure, trigger the error state: "No data".
- When navigating from one page to the other or opening the email for example, present the loading icon.
- On the Daily Action Queue page add a button to clear the search bar or clear filters
- On the Daily Action Queue, create the path, which path is it inside the whole CRM and create just a mock of the menu so that it does not look lik

Maintain the same design language throughout and tether all behavior strictly to these rules.
```

### Step 3: Refine, one surgical polish
```
The Daily Action Queue page needs a professional style polish.
1. The biggest unprofessional look is the filtering styles and section. 

Don't change anything else in the project or touch the underlying logic.
```

## Reusable techniques learned

- Break the interaction with the LLM into different PROMPTS
- Group the changes by like area: the visual polish, the behavior, etc.
- When possible just request one change and QA that one, instead of grouping a lot of requests into one

## What broke (and the fix)

-Single mega-prompt tends to fail terribly.
-If you don't provide a visual template, you might end up iterating and wasting tokens unnecessarily.

_____
