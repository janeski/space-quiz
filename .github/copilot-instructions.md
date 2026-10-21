# Project constraints

Orbit is a standalone space exploration quiz. Keep the application in a single
`index.html`, including CSS, JavaScript, SVG artwork, and decorative effects.
It must run directly from a `file://` URL without a server, dependencies,
external fonts, or network requests.

## Running and checking

Open `index.html` directly in a browser or the integrated browser. From
PowerShell, `Start-Process .\index.html` opens the default browser.

For a focused browser check, answer one question correctly and confirm one point,
green feedback, an explanation, locked answer buttons, enabled Next, and 10%
progress. Answer the next question incorrectly and confirm the score stays the
same, the selected answer turns red and shakes, the correct answer is revealed,
and progress advances to 20%.

For a full smoke test, finish the quiz with scores of 0, 7, 8, and 10. Check final
score, accuracy, accessible emoji reaction, all ten review entries, and replay
resetting progress, score, review expansion, and celebration. Celebration begins
at 8, independently of the rank thresholds.

Run keyboard checks with real Tab, Shift+Tab, Enter, and Space events, including
answer selection, Next, opening and closing the review, and replay. Calling
`element.click()` or dispatching synthetic DOM keyboard events is not a substitute
for native keyboard verification. Check both color schemes, reduced motion,
and a 320px viewport as well as desktop.

### Browser automation coverage

Navigate to the absolute `file://` URL for the current checkout, then reload
after edits. Test the actual page, not a reconstructed DOM. Capture uncaught
page errors and inspect the rendered UI as well as DOM assertions.

Use the following observable contracts for targeted tests:

| Scenario | Expected behavior |
| --- | --- |
| Fresh load | First question, four enabled answers, zero points, Next disabled, results hidden |
| Answer submitted | All four answers disabled; only one response recorded; repeated activation cannot add points |
| Correct answer | Selected button has `.correct`, score increases by one, feedback includes the question's fact |
| Incorrect answer | Selected button has `.wrong`, correct button has `.correct`, score unchanged, explanation names the correct answer |
| Next | Question changes, old feedback clears, answers unlock, Next disables; final question offers See results |
| Progress | `aria-valuenow` equals answers submitted; fill width and percentage agree, reaching 100% on the tenth answer |
| Results | Quiz hidden, results visible; final score and accuracy agree; review contains ten entries matching chosen answers and facts |
| Celebration boundary | No celebration at 7; visible celebration at 8; perfect-score trophy at 10 |
| Replay | First question restored with zero points and progress; results hidden; review closed; celebration cleared |

For keyboard automation, use the browser driver's native key-press capability.
Starting from a fresh navigation, Tab reaches the first answer; subsequent Tabs
reach the other answers in order while disabled Next is skipped. Exercise both
Enter and Space for answers. Verify focus moves to Next after submission, to
the question heading after advancing, and to the result heading on completion.
From the result heading, Tab reaches Launch again and then the review summary;
Space/Enter toggles the review and Shift+Tab returns to Launch again.
Check `:focus-visible` and a visible outline on each interactive target.

Emulate light and dark color schemes and `prefers-reduced-motion: reduce`
through browser media settings. In reduced motion, the star-field
pseudo-elements, answer feedback, and confetti must have no active animation;
the static correct/incorrect states and celebration must still convey meaning.
At 320px and desktop widths, check for horizontal overflow and focus visibility.
DOM assertions alone do not establish screen-reader announcement quality;
retain the live-region semantics and report separately whether assistive
technology was actually used.

## Application structure and state

`index.html` contains theme/layout rules, the intro and mission-control panel,
mutually exclusive quiz/results cards, and a final strict-mode script.
The script runs after the markup; `$` is a local `getElementById` helper.

The `questions` array is the content source: each entry has `category`,
`question`, four `answers`, a zero-based `correct` index, and an explanatory
`fact`. Keep ten questions. Some counts are explicitly present in markup,
including intro copy and `aria-valuemax`; changing the array alone does not
update every surface.

State is held in `current` (zero-based question index), `score`, `selected`
(`null` before answering), and `responses` (chosen indices in question order).
`renderQuestion()` creates fresh answer buttons and gates Next.
`selectAnswer()` guards against repeat scoring, locks all answers, reveals
feedback, and updates progress. Progress measures answered questions, not
the currently displayed question. `showResults()` builds the scorecard and
review. The restart handler resets both state and presentation.

## UI and interaction conventions

- Preserve the centered 540px maximum-width column, sans-serif body text,
  and monospaced mission-control accents. Styling uses CSS custom properties
  from `:root`; dark-mode overrides also include mission-local tokens and
  SVG colors.
- Keep automatic `prefers-color-scheme` support. New animation must remain
  meaningful when disabled by the final `prefers-reduced-motion` rule, which
  covers pseudo-elements, transitions, and scrolling.
- Answer states use `.correct`, `.wrong`, and `.dimmed`; feedback uses
  `.incorrect`. Correct answers have a check mark, incorrect answers a cross,
  and explanations convey the outcome without relying on color or animation.
- Results ranks are Commander at 10, Astronaut at 7-9, Cadet at 4-6, and
  Stargazer at 0-3. `.celebrating` and `#celebration` are enabled only at 8-10;
  replay clears both.
- Preserve native buttons and `details`/`summary` keyboard behavior. After
  answering, focus moves to Next; advancing or restarting focuses `#question`;
  finishing focuses `#result-title`. Headings use `tabindex="-1"` for
  programmatic focus without entering the normal Tab order.
- Retain visible focus rings, score/feedback live regions, the labeled
  progressbar and its numeric ARIA value, and emoji accessible labels.
  Decorative confetti is `aria-hidden`; decorative background layers must
  not intercept pointer input.
- Dynamic question, answer, and explanation text is assigned through
  `textContent` and DOM creation. Existing `innerHTML` usage is limited to
  controlled templates for counters and button labels.
- Visibility uses the `hidden` attribute with `[hidden] { display: none
  !important; }`; do not override it with component display rules.
