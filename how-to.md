# TrueFIRE — Technical How-To

## Project structure

```
truefire/
├── index.html                  ← the engine (never edit for content changes)
└── data/
    └── topics/
        ├── topics-index.json   ← registry: tells the engine what topics exist
        ├── monetary-policy.json
        ├── demand-elasticity.json
        └── market-structures.json
```

The engine (`index.html`) and the content (`data/topics/`) are **completely separate**. The engine fetches data at runtime — it has no knowledge of Economics, Biology, or any subject. You add subjects by dropping JSON files. You never touch `index.html` for content changes.

---

## Running locally

The app must be served over HTTP — it uses `fetch()` to load JSON files, which browsers block on `file://` URLs.

The simplest option (requires Node.js):

```bash
npx serve truefire/
```

Then open `http://localhost:3000` in your browser. Any other static server works fine (`python -m http.server`, VS Code Live Server, etc.).

---

## Adding a new topic

Two steps.

**Step 1 — Create the JSON file**

Create `data/topics/your-topic-id.json`. Use lowercase-with-hyphens for the filename. The schema:

```json
{
  "id": "your-topic-id",
  "title": "Display Title",
  "description": "Short subtitle shown on the topic card",
  "questions": [
    {
      "statement": "The statement the student evaluates.",
      "answer": true,
      "payload": "Terse explanation. Use <strong>bold</strong> to highlight the pivot word."
    },
    {
      "statement": "Another statement.",
      "answer": false,
      "payload": "Correction. One punchy sentence that identifies the exact point of failure."
    }
  ]
}
```

**Step 2 — Register it in the index**

Open `data/topics/topics-index.json` and add one entry:

```json
[
  { "id": "monetary-policy",   "title": "Monetary Policy",   "description": "Central bank tools, interest rates, money supply" },
  { "id": "demand-elasticity", "title": "Demand & Elasticity", "description": "Price elasticity, total revenue, demand curves" },
  { "id": "market-structures", "title": "Market Structures",  "description": "Perfect competition, monopoly, monopolistic competition" },
  { "id": "your-topic-id",     "title": "Your Title",         "description": "Your subtitle" }
]
```

Reload the app. Your topic appears in the list. No code changes.

---

## Content guidelines

**The terse payload rule:** one punchy sentence that identifies the exact misconception and replaces it with the truth. Use `<strong>` tags to highlight the pivot word.

| Style | Example |
|---|---|
| Too long | "False. While the Central Bank can use open market operations, purchasing securities actually injects liquidity, thereby expanding the money supply through the banking system." |
| Too empty | "Incorrect. The answer is False." |
| Correct | `"False. Buying bonds <strong>increases</strong> the money supply, whereas selling them decreases it."` |

Two sentences are fine when a concept genuinely needs it. Never truncate a good explanation to hit a length target.

**Question balance:** No cap on question count. The engine renders whatever is in the array and shuffles on every session, so rote memorisation isn't a concern.

**HTML in payloads:** Only `<strong>` tags are needed. Avoid anything else — keep payloads as plain text with one bolded pivot word.

---

## How the engine works (overview)

On load, the engine fetches `topics-index.json` and renders the topic list. When a student taps a topic, the engine fetches that topic's JSON file and starts the drill. All session state (streak, score, answers) lives in memory and resets on each new session.

The 3-step drill loop:

1. **Blind statement** — student sees the statement and two buttons (True / False). Nothing else.
2. **Verdict** — screen flashes green or red. Verdict badge and payload slide in.
3. **Next** — if correct, "Next" is immediately tappable. If wrong, "Got it" is locked for 2.5 seconds to force a brief moment of reflection.

After all questions, the session summary shows score, best streak, and a scrollable review of every question with the student's answer highlighted.

---

## Deploying

The app is a static site — no backend, no database. Any static host works.

**Netlify (simplest):** drag the `truefire/` folder into [netlify.com/drop](https://app.netlify.com/drop).

**GitHub Pages:** push the repo and enable Pages on the `main` branch. Set the root to `/` (or `/docs` if you rename the folder).

**Vercel:** `vercel truefire/` from the CLI.

No build step. No configuration files needed.

---

## Adding a new subject (e.g. Biology)

Same two steps as above, repeated for each topic within the subject.

The `topics-index.json` can hold any number of entries across any number of subjects. If you want to group them (Economics / Biology / History), that's a v2 UI concern — the data format already supports it since each topic carries its own `id`, `title`, and `description`.

---

## Common errors

| Error | Cause | Fix |
|---|---|---|
| "Couldn't load topics" on boot | App opened as `file://` instead of `http://` | Run `npx serve truefire/` and open `localhost:3000` |
| Topic card appears but drill won't start | JSON filename doesn't match the `id` in `topics-index.json` | Make sure `"id": "my-topic"` matches the filename `my-topic.json` |
| Payload bold not rendering | Used `**bold**` (Markdown) instead of `<strong>bold</strong>` (HTML) | Use HTML tags in the `payload` field |
| New topic not appearing | Entry added to JSON file but not to `topics-index.json` | Add the entry to the index — both steps are required |
