---
name: asd
description: Write or rewrite prose in ASD-STE100 Simplified Technical English (STE), the controlled English of aerospace and defense maintenance documentation. Use when the user types /asd, or asks for Simplified Technical English, ASD-STE100, STE, or controlled-language phrasing. Bare /asd restates the previous assistant message in STE. With a fuller request, apply STE to whatever that request points at — the next reply, a document, a README, code comments, UI strings, release notes.
---

# ASD-STE100 Simplified Technical English

## Step 1: decide the scope

**Mode A — bare `/asd`** (no arguments, or arguments that only say "do it").

Restate your own most recent message in STE. Nothing else:

- Do not answer a new question, run tools, or redo work.
- Do not add, remove, or soften facts. Same content, controlled language.
- Output the rewritten text only. No preamble, no "here is the STE version".
- Leave code blocks, commands, paths, identifiers, and quoted output verbatim. STE governs prose.
- If the last message was mostly tool output, rewrite the prose around it.

**Mode B — `/asd` with context, or STE requested inside a larger message.**

Read the message to find *what* gets STE and *when*. Then apply it to that target only; the rest of your reply stays normal English.

| User says | STE applies to |
|---|---|
| "/asd explain how the auth flow works" | your next reply |
| "/asd write the install section of the README" | the document you write, not your chat around it |
| "/asd for the docstrings in this module" | the docstrings only; code unchanged |
| "rewrite these warning strings in STE" | those strings |
| "/asd from now on" | your prose for the rest of the session |
| "/asd — is this doc compliant?" | a review, not a rewrite: list the violations with fixes |

If the scope is genuinely ambiguous, pick the most likely target, apply STE, and say in one short line what you applied it to.

## Step 2: apply the rules

STE has nine rule sections. The working set:

### Words

- One word, one meaning. One meaning, one word. Never swap in a synonym for variety — if it is a `valve` in step 1, it is a `valve` in step 9.
- Use the shortest, most common, most concrete word that carries the meaning. See `references/word-choices.md` for the substitutions that come up most.
- Use an approved word only in its approved part of speech. Some words are nouns only, so the verb becomes a verb + noun pair: `Do a test of the pump`, not `Test the pump`.
- Technical names are allowed: part names, tool names, materials, standards, numbers, units, place names, product names. Keep the official nomenclature exactly.
- Technical verbs are allowed when no plain verb works: `drill`, `solder`, `crimp`, `ream`, `torque`.
- No idioms, slang, metaphor, humor, or jargon. `Kick off the build` → `Start the build`.
- Write out `for example` and `that is` instead of `e.g.` and `i.e.` Expand every abbreviation at first use unless the project has an approved list.
- Requirements use `must`. Possibility and permission use `can`. Prohibition uses `do not`. Avoid `shall`, `should`, `may`, `might`.

### Noun phrases

- Maximum three words in a noun cluster. Break longer ones with prepositions or hyphens: `main landing gear door actuator seal` → `seal of the actuator for the main landing gear door`.
- Keep the article or demonstrative: `Remove the bolt`, not `Remove bolt`.
- Hyphenate words that act as one modifier: `a two-stage pump`.
- Do not delete words to make text shorter. Telegraphic style is a violation, not a virtue.

### Verbs

- Use the imperative, the infinitive, the simple present, the simple past, and the simple future.
- No perfect or continuous tenses. `The valve has been closed` → `The valve is closed` or `You closed the valve`.
- No `-ing` form as a noun or an adjective, unless the word is itself approved or part of a technical name (`bearing`, `wiring`, `O-ring`). `Before installing the pump` → `Before you install the pump`.
- `The following steps` → `The steps that follow`.
- Active voice always in procedures. Active voice as much as possible in description; passive only when the actor is unknown or irrelevant.

### Sentences

- Procedural sentence: 20 words maximum. Descriptive sentence: 25 words maximum.
- One instruction per sentence. Two actions in one sentence only when they happen at the same time.
- Keep the natural order: subject, verb, object. Keep related words together.
- Put the condition before the instruction: `If the lamp comes on, close the valve.`
- Descriptive paragraph: 6 sentences maximum, one topic, topic sentence first.

### Procedural writing

- Give instructions in the imperative: `Turn off the power supply.`
- One step, one action. Number the steps.
- Use a vertical list when the text has parallel actions, conditions, or items.

### Safety instructions

- The warning or caution goes **before** the step it applies to, never after.
- Start with a short command, then the consequence: `Do not touch the terminals. High voltage can kill you.`
- `WARNING` = injury to people. `CAUTION` = damage to equipment. Keep both short.

### Punctuation

- A colon introduces a vertical list.
- Do not use the slash for `and` or `or`. Keep it only in units such as `km/h`.
- Do not use `&`. Write `and`.
- Keep parentheses out of instructions; make the aside a separate sentence.

## Step 3: check before you answer

Run through this list once:

1. Any sentence over 20 words (procedure) or 25 words (description)?
2. Any `-ing` noun or adjective?
3. Any perfect or continuous tense?
4. Any passive voice in a procedure?
5. Any noun cluster over three words?
6. Any missing article?
7. Any word chosen for elegance where a plainer one exists?
8. Any term that changed name between sentences?
9. Any warning placed after its step?
10. Any idiom or abbreviation that a non-native reader would miss?

## What STE is not

STE controls the language, not the engineering. Do not lose a qualifier, a number, a condition, or a caveat to make a sentence shorter — split the sentence instead. Do not translate identifiers, error strings, API names, or CLI flags. The result should read as flat and unambiguous, not as writing for children.

Note on the dictionary: ASD-STE100 approves roughly 900 general words, and the full list ships with the specification from the AeroSpace and Defence Industries Association of Europe. Use `references/word-choices.md` and the heuristics above; when a word is not certainly approved, prefer the plainest common alternative, and say so if the user needs strict compliance for a controlled document.
