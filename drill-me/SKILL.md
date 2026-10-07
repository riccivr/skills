---
name: drill-me
description: "The reverse of grill-me. Interrogate the user about a feature, epic, bug, ticket, PR or subsystem that you have already read up on, so they learn it through retrieval practice instead of reading a summary. You hold the answer key; they do the talking. Features a full questionnaire mode with interactive multi-question pickers in Claude Code CLI and Desktop. Concludes with an overall score, scoring reasoning for each question, and a domain explanation debrief. Use whenever the user says drill me, quiz me, test me, questionnaire, make sure I understand, help me soak up or get my head around a ticket / epic / PR / module, or is about to pick up work that someone else shaped. Not for explaining something (that is a walkthrough) and not for stress-testing the user's own plan (that is grilling)."
argument-hint: "[ticket key | PR number | path | nothing for the current branch] [--questionnaire] [--quick | --deep]"
---

# Drill me

The user is about to work on something they do not yet hold in their head. An epic someone else shaped, a bug report, a PR they have to review, a subsystem they have never touched. They could ask you to explain it. Reading an explanation feels like learning and mostly is not. Retrieving an answer from memory, getting it wrong, and being corrected is what makes it stick.

So learn the subject yourself, then make the user produce it. You hold the answer key. They do the talking.

This is the mirror of `grilling`. There, finding facts is the model's job and making decisions is the user's. Here, finding facts is still the model's job, and recalling them is the user's. You never ask the user to decide anything, and you never ask something you have not verified.

## 1. Learn it first, silently

Before asking a single question, know the subject well enough to grade any answer with a citation. Resolve the argument.

- **Issue key.** Pull the whole ticket, every section, the comments, the attachments, through whatever tracker tool is available (an MCP, a CLI, the web). A ticket's hidden fields and comment thread usually hold the decisions the description leaves out.
- **PR number.** The diff, the description, and the review threads. Review threads are where the reasoning nobody wrote down elsewhere lives.
- **Path.** The code and its tests. Tests state intent more plainly than the implementation does.
- **Nothing.** The current branch against its base, plus the ticket whose key appears in the branch name, if there is one.

Then read the code the subject touches, and follow it outward to the callers and consumers that show what it is for. If a codebase-walkthrough skill is available (`how`, for instance), use its exploration step and withhold the explanation. Check any memory or notes directory for prior context on the subject.

Now build the **concept map**, the list of load-bearing concepts. The test for load-bearing is "if the user did not know this, what would they get wrong?" If the answer is nothing, it does not go in the bank. Aim for five to ten concepts. Order them by dependency, because you cannot ask why a thing exists before the user knows what it is.

Every concept gets an answer key and a citation: a `file:line`, a ticket section, a PR comment. **If you cannot cite it, you cannot ask it.** The whole exercise rests on your read being right. A wrong answer taught with confidence is worse than no drill at all. When you are unsure about something, drop it from the bank and mention it at the close as a thing worth checking.

Tell the user only the subject, the number of concepts, and roughly how many questions to expect. Do not list the concepts. The list is the answer key.

## 2. Ask one question at a time

One question per turn. Grilling asks a whole frontier in one round. This does not, on purpose. Rounds suit decisions because decisions are independent of each other. Learning is not. Working memory is small, and each answer should shape the next question.

Each concept climbs a ladder: **what** it is, **why** it exists, **how** it works, **what if** it is bent (an edge case, a failure mode, a change). Start at "what" for the root concept. Two clean hits in a row at a level means skip up a level or on to the next concept. A miss means stay at that level and hint.

### Question formats

Questions can be open-ended or multi-option:

- **Standard format.** Open-ended prompt where the user explains the answer in their own words.
- **Questionnaire mode (`--questionnaire`).** A full multi-option quiz format. Every question gives the user choices to pick from.

When asking multi-option questions:

- **Interactive tools in Claude Code CLI and Desktop.** If an interactive question tool is available (`AskUserQuestion` in Claude Code, `ask_question` in Antigravity), invoke it. Claude Code CLI and Claude Desktop present an interactive picker where the user selects an option using arrow keys or a click.
- **Text fallback.** When no interactive question tool is available, format the choices as a labeled list (A, B, C, D) directly in the turn and prompt the user to pick one.
- **Option quality.** Write three to four options of balanced length and detail. Never make the correct answer stand out through formatting or verbosity.
- **Plausible distractors.** Draw distractors from real pitfalls, adjacent subsystems, or common misconceptions in the codebase. Never include joke options.
- **Write-in answers.** Allow the user to type a custom answer if they disagree with all options or want to elaborate.

### Question rules

- **Never leak the answer in the question.** Naming the mechanism you are about to ask about, or narrating the context up to the answer, turns retrieval into reading.
- **Ask about behaviour, causes and consequences.** Never about identifier names, line numbers or exact signatures. Trivia tests memory of text, and the user can grep for text.
- **No opinion questions.** "Would you have designed it differently?" belongs in grilling.
- **No stacked questions.** Two questions in one turn get one answered.

Format for open-ended questions:

```
Q3 (why). <the question, one to three sentences>
```

Format for multi-option questions (when not using an interactive tool):

```
Q3 (why). <the question, one to three sentences>

A) <option 1>
B) <option 2>
C) <option 3>
D) <option 4>
```

A bad question. "What does `resolveOfferLetterTexts` do?" It leaks the name, and knowing what a function does from its name is trivia.

A good question for the same concept. "A staff member edits a condition that contains a template token and then issues the letter. What ends up in the PDF, and at what point does that get decided?" The user has to produce the mechanism and place it in the flow.

## 3. Grade every answer the same way

```
✅ right  |  🟡 partly  |  ❌ wrong
<correction, at most three sentences, with the citation>

Q4 (how). <next question>
```

- **Right.** One line at most. No praise, no "and also". Praise and elaboration are both you talking when they should be.
- **Partly.** Name the missing piece, not the whole answer.
- **Wrong.** The correct answer with its citation. In questionnaire mode, note why the selected option failed. Then **requeue the concept** and ask it again later in the session in a different form, a what-if instead of a how. One correction does not fix a wrong answer. A second retrieval later does.
- **"Don't know", first time.** A hint, not the answer. Narrow the space, point at a file to open, or narrow down the options.
- **"Don't know", second time.** The answer, then requeue.
- **The user disagrees with your key.** Go back to the source before insisting. If they were right, say so and fix the map. Your read is the answer key only as long as it survives contact with the code.

Keep your text short. If a turn has more of your words than the user's, the drill is turning into a lecture. Save the full domain explanations and detailed scoring breakdown for the close.

## 4. Modes and depth

### Modes

- **Standard (default).** Open-ended questions that force active recall from scratch. Best when preparing to write implementation code.
- **Questionnaire (`--questionnaire`).** A full multiple-choice mode. Every turn presents an interactive multi-option question using Claude Code CLI and Desktop's native picker (`AskUserQuestion`) or `ask_question`. The user selects one option to proceed. Best for fast knowledge checks, architecture refreshers, or reviewing PRs without typing long responses.

### Depth

- `--quick` asks five questions on root concepts only. Enough for standup.
- The default asks around ten questions, or stops when every concept has passed once, whichever comes first.
- `--deep` runs until every concept has passed at the what-if level, requeues included. For the thing the user is about to build or review.

Modes and depth combine freely: `/drill-me --questionnaire --quick` runs a five-question interactive questionnaire.

Stop early if the user asks. Still do the close.

## 5. Close

Finish with the Feynman test. Ask the user to explain the whole subject back in five sentences, as if to a colleague picking it up tomorrow. Check it against the concept map for what is missing, what is wrong, and what is right but in the wrong order.

Then produce the final session report with an overall score, a question-by-question breakdown explaining every score and its domain context, and the cheat sheet.

### Scoring rules

- **Right (1.0):** User retrieved the core mechanism cleanly or picked the correct option on the first ask.
- **Partly (0.5):** User retrieved part of the mechanism, picked an option after a hint, or needed a second attempt.
- **Wrong (0.0):** User missed the concept, picked an incorrect option in questionnaire mode, or gave up. Requeued attempts that pass can earn back up to 0.5 for recovery, but note the initial miss.
- Calculate the final score as total points earned divided by total points possible, expressed as both a fraction and a percentage. Include a one-sentence assessment of readiness to touch the code.

### Report format

```
## Drill: <subject>

Overall score: <X/Y> (<percentage>%)
Readiness: <one sentence assessment based on the score and missed concepts>

### Breakdown

#### Q1 (<level>): <question summary or concept name>
- **Score:** ✅ Right (1.0/1.0) | 🟡 Partly (0.5/1.0) | ❌ Wrong (0.0/1.0)
- **Score reasoning:** <why this score was awarded, including the option selected if in questionnaire mode and why it was right or wrong>
- **Domain explanation:** <the model's thorough explanation of how this part of the domain works, the underlying mechanisms, why it was built this way, and citations (file:line, ticket, or PR)>

[Repeat for every question asked]

### Summary

Missed first time: <concepts, one line each, with citations>
Got straight away: <concepts>
Cheat sheet: <every concept in dependency order, one line each, with citations>
Not asked because I wasn't sure: <anything dropped from the bank, if any>
```

The close is where full explanations belong. They land on top of retrieval instead of replacing it. Where a persistent memory or notes directory exists, offer to record the missed concepts so a later drill on the same subject starts where the user was weak.

## What this skill is not

- **Not a walkthrough.** If the user wants the explanation, give them the explanation and skip the drill. Do not smuggle a lecture in as a series of "corrections".
- **Not grilling.** You are not extracting the user's thinking or sharpening their plan. The knowledge already exists in the source. Your job is to get it into the user's head.
- **Not a code review.** If you spot a defect while learning the subject, note it for the close. It is not a question.
