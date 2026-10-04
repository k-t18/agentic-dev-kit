# Run setup

Setup differs from project to project and from day to day. Ask at the start of **every**
run — never reuse a URL or a backend answer from an earlier run or from a config file.

## 1. The browser tools

Before any question, check that the session exposes the Claude in Chrome tools
(`mcp__claude-in-chrome__*`). If it does not:

```
Flow test cannot run: the Claude in Chrome tools are not available in this session.
Install or connect the Claude in Chrome extension, then run
/agentic-dev-kit:flow-test <flow-name> again.
```

Then stop. Do not read the code and guess the results.

If the tools exist but Chrome does not respond, say that and ask the developer to open
Chrome and check the extension. Do not retry in a loop.

## 2. The three questions

Ask them together, in one message, and wait.

| # | Question | Why it matters |
| --- | --- | --- |
| 1 | What URL is the web app running at? | The only origin the run will drive. |
| 2 | What is it pointed at — a real test backend, or mocked APIs? If mocked, which APIs and how? | Decides what is safe to do and what a result proves. |
| 3 | How does sign-in work for this app, and which test account or role should the run use? | The developer signs in; the run needs to know when it is done and what that user can see. |

Follow-ups, only when the answer leaves them open:

- **Real backend:** is it a shared environment or the developer's own? Is there test
  data the run should use, and anything it must not touch?
- **Mocked:** are all the flow's APIs mocked, or only some? An API with status
  `pending` in the spec that is not mocked makes its criteria untestable.
- **A production URL or a production backend:** say that this run changes data, and ask
  the developer to confirm in so many words before going on. Prefer a test environment.

Record the answers. They go at the top of the report.

## 3. Is the app running?

Open the URL in a **new tab**. Leave the developer's other tabs alone.

- The page loads → continue.
- It does not load → say what was seen (connection refused, error page, blank page) and
  ask the developer to start the app. **Do not start a dev server, a mock server, or
  any other process yourself** unless the developer asks for it in this conversation.

## 4. Sign-in

The developer signs in, in Chrome, by hand.

1. Tell them which tab the run is using and ask them to sign in there.
2. Wait until they say they are signed in.
3. Confirm by reading the page — a signed-in screen, not the sign-in form.

Never:

- type a password, a one-time code, a PIN, a recovery answer, or an API key into any
  field;
- read a credential from `.env`, a config file, a fixture, or the page and enter it;
- complete a CAPTCHA or any other bot check;
- approve a consent or permission screen from an identity provider — ask the developer.

If the session expires in the middle of the run, stop, say which criterion was in
progress, and ask the developer to sign in again. Run that criterion again from its
Given.

**When the flow under test is itself a sign-in or verification flow:** the steps that
enter credentials are done by the developer while the run observes the result. Mark the
plan row "developer performs step N". If the developer would rather not, the criterion
is `Skipped` and goes on the list for a human.

## 5. Before the first criterion

- Read the console and the network once, so errors that were already there are known and
  are not blamed on a criterion.
- Note the window size the run uses. If the spec or the developer names a size, set it.
- State the starting point back to the developer in two lines: URL, backend, signed in
  as which role, spec path and status.
