# Evidence

A result without evidence is an opinion. Every criterion gets evidence from three
places; a failure's evidence is printed in full.

## The three sources

| Source | Collect | For |
| --- | --- | --- |
| **Page** | The route; the text or element the Then names, as read from the page; the state of the control acted on. A screenshot when the session can take one and the failure is visual. | Every criterion |
| **Console** | Errors and warnings logged between the When and the observation. | Every criterion |
| **Network** | Requests sent between the When and the observation: method, path, status. Which `API-` ID each matches. | Every criterion |

Record observations as what was seen, not what it means. "The button stayed disabled
after all four fields were filled" — not "validation is broken".

## Attributing console and network entries

- Compare against the starting point noted before the When (`executing-criteria.md`,
  step 3). Only what appeared after it belongs to this criterion.
- Entries that were already there before the first criterion are background. Report
  them once, under incidental findings, not against every criterion.
- Noise that is not the app's — browser extension messages, development-only notices
  from the framework, hot-reload chatter — is left out. When unsure, include it and say
  it may be noise.

## What counts as a finding even when the criterion passed

| Seen | Report as |
| --- | --- |
| A console error | Incidental — console |
| A request that failed (4xx, 5xx, or no response) and the spec does not say it should | Incidental — network |
| A request the spec does not list in section 7 that is clearly part of the flow | Question for the spec |
| The same request sent twice for one action | Incidental — network |
| A value section 11 says must be masked, shown unmasked | `Fail` if a criterion covers it; otherwise incidental, flagged as sensitive |

An expected failed request is not a finding: a criterion that checks an error path is
supposed to produce the error response in section 7.

## What must not go into evidence

- Passwords, one-time codes, tokens, API keys, session cookies, `Authorization` or
  `Cookie` header values — never, even partly.
- Real personal data. Anything section 11 lists is written as a placeholder
  (`<phone number>`, `<document number>`), not copied from the page or a response body.
- Full request or response bodies. Name the field that matters and its type or a
  placeholder value.
- Query strings that carry tokens or personal data — give the path, and `?…` for the
  rest.

A screenshot that shows personal data is described in words instead, and is not attached
to a PR comment.

## Reproduction steps for a failure

Written so a person can follow them with no context:

```
AC-04 — FAIL
Given <state>, when <action>, then <expected result>.   Covers: F-03, R-02, E-01

Steps
  1. Signed in as <role>, open <route>.
  2. <action> with <test data placeholder>.
  3. <the When>.

Expected   <the Then, with the exact message from E-01>
Observed   <what the page showed instead>
Console    <error text, first line> (<source file>:<line> if shown)   | none
Network    POST <path> → 500 (API-02)                                  | none
Notes      <anything that narrows it: happens only on the second attempt, etc.>
```

- Number the steps. Start from a state anyone can reach — signed in, at a route.
- Quote the expected result from the spec and the observed result from the page.
- Do not add a cause or a fix. Where the evidence points somewhere obvious (a request
  returned 500; a console error names a component), state the evidence and stop there.
- If the failure did not happen again on a second attempt, say so: `seen 1 of 2 runs`.

## Evidence for `Could not test`

One line: how far the run got and what stopped it. If a console error or failed request
appeared at the point it stopped, include it.
