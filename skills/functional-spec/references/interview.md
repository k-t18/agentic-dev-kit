# The interview

**Do not write the spec yet.** The interview decides what goes in it.

## Rules

- **One round per message.** Ask the round's questions, wait, then move on.
- **Skip what is already answered** — by the PM's opening description, by Figma, or by
  the codebase. Say what you took from where, so the PM can correct it.
- **Ask about the specific thing, not the category.** "What happens if the address
  check fails?" — not "any error cases?".
- **Play back before moving on.** End each round with a two-line summary of what you
  now understand.
- **"I don't know" is a valid answer.** Record it as an open question (`Q-nn`) with who
  must answer and whether it blocks approval. Never supply the answer yourself.
- **Do not suggest answers to business questions.** Offering options for the PM to pick
  from is fine only when the options come from Figma or existing code.

## Rounds

| # | Round | Fills section |
| --- | --- | --- |
| 1 | Why | 2 Purpose |
| 2 | Scope | 3 Scope |
| 3 | The flow, screen by screen | 4 Screen inventory, 5 Flow steps |
| 4 | Branches and rules | 6 Branches and rules |
| 5 | APIs and data | 7 API contracts, 8 Data and offline |
| 6 | Offline | 8 Data and offline |
| 7 | Errors and screen states | 9 Screen states, 10 Validation and errors |
| 8 | Sensitive data | 11 Sensitive data |
| 9 | Native differences (`web+native` only) | 12 Native notes |

### 1 — Why

- Who is the user, and what are they trying to get done?
- What do they do today without this flow?
- How will we know the flow works — what is the successful end state?

### 2 — Scope

- Where does the flow start, and what are all the places it can end?
- What is explicitly **not** part of this flow, even though it is nearby?
- Does it depend on another flow or feature being finished first?

### 3 — The flow, screen by screen

Start from the screen list (`figma-screens.md`). For each screen, in order:

- What can the user do here?
- For each action: what does the system do, and where does the user land next?
- Is there a way back? What is kept and what is lost when they go back?

A screen in Figma that no step reaches, or a step with no screen, is a question to ask —
not something to resolve yourself.

### 4 — Branches and rules

For every point where the flow can go more than one way:

- What decides which way it goes — and where does that fact come from (an API, stored
  data, the user's choice)?
- What happens on **each** outcome, including the negative one?
- Are there different kinds of user who see different things?
- Can a step be skipped, repeated, or resumed later?

### 5 — APIs and data

For every step where the system reads or saves something:

- Does the API exist? If yes, ask for a real curl or request/response sample.
- If it does not exist yet: what must go in, what must come back, and who is building it?
- What does the API return when it refuses the request?

Record each API with a status (`ready` / `pending` / `unknown`). Only a pasted real
sample earns `ready`.

### 6 — Offline

- Must any part of this flow work without a connection?
- If the user saves something while offline: is it queued and sent later, or blocked?
- What should the user see while a queued save has not been sent yet?
- If a queued save is rejected by the server later, what happens?

If the whole flow needs a connection, record that as a rule and what the user sees
when there is none.

### 7 — Errors and screen states

For each screen: what does the user see while it loads, when there is nothing to show,
when it fails, and when an action is not allowed yet?

For each input: what makes it invalid, and what message appears?

### 8 — Sensitive data

- Does the flow handle personal data, identity documents, payment details, or secrets?
- May any of it be kept on the device, and for how long?
- Is anything shown masked, or hidden after first entry?

### 9 — Native differences

Only for `web+native`. Which steps rely on something a phone does differently from a
browser — camera, file picking, permissions, deep links back into the app, third-party
SDKs, or behaviour when the app is backgrounded?

## Ending the interview

Stop when every round is either answered or turned into open questions. Then say how
many open questions there are and which ones block approval, and move to the draft.
