# LLM JSON Repair

Paste the broken JSON an LLM just handed you, get valid JSON back, plus a named diagnosis for every repair and the prompt change that prevents it next time.

**Live demo:** https://0xelitesystem.github.io/llm-json-repair/

Not a general JSON tool: [json-workbench](https://github.com/0xelitesystem/json-workbench) already covers formatting, tree browsing, querying, diffing and everyday tolerant parsing, while this repo is specifically about LLM output failure modes and the prompt-level fix for each one.

## Use

1. Paste the model's response into the Model output box, or click Load sample.
2. Click Repair and diagnose, or press Ctrl or Cmd + Enter.
3. Read each named finding, where it applied, and its "prevent this" note.
4. Click Copy pretty, Copy minified or Copy report.

## Why this exists

Models regularly return JSON wrapped in code fences, with trailing commas, or cut off mid-object, and the usual fix is a silent repair that hides what went wrong. This is a single HTML file with no tracking and no network calls that repairs the output, names every fix and points at the prompt change that prevents it, MIT licensed.

## Features

- **A tolerant JSON parser written from scratch in the page.** No external dependencies, no library, no build step. It parses what the model actually produced instead of what JSON allows.
- **Every repair is a named finding**, with the line and character offset it applied at, a one-line explanation of what was wrong, and a "prevent this" note giving the concrete prompt or API change that stops that failure at the source.
- **24 named failure modes**, including: wrapping markdown code fences (with or without a language tag), prose before the JSON, a stray trailing sentence after a complete object, trailing commas, missing commas between members, missing colons, single-quoted strings, curly quotes used as string delimiters, unquoted object keys, bare word values, Python `True` / `False` / `None`, JavaScript `NaN` / `Infinity` / `undefined`, wrong-case literals, `//` and `/* */` comments, duplicate keys, unescaped newlines or tabs inside strings, invalid escape sequences, a leading plus sign or leading zeros in numbers, invisible whitespace, more than one JSON document in one response, and truncation.
- **Truncation is handled and flagged, not hidden.** If the output was cut off mid-string, mid-array or mid-object, the tool closes the containers that were still open, reports exactly where the cut happened, and states plainly that the missing content is unrecoverable.
- **Verification step.** The repaired document is run back through `JSON.parse`, and the result reports that it parsed plus the top-level type and key or element count. Output that does not parse is never presented as repaired.
- **Before and after panes**, a pretty-printed result with a copy button, a minified copy button, and a copy-as-markdown diagnostic report.
- **Honest states.** Valid input reports "no repairs needed" and changes nothing. Input too damaged to interpret gets a failure card with the failing line, a caret pointing at the character, and an explanation of what could not be recovered, rather than confidently wrong output.
- **Duplicate keys are caught even when the document is already valid.** `JSON.parse` accepts a repeated key and silently keeps the last value, so a strictly valid response can still have lost data. That case is reported as a finding instead of being waved through as clean, and nothing is rewritten.
- **Load sample** fills the box with a deliberately awful model response that trips 15 distinct failure modes at once, truncation included.
- Dark theme by default with a light toggle that persists, responsive down to 360px, keyboard accessible, `Ctrl` or `Cmd` + `Enter` to run.

## How it works

The pipeline is four stages, all client side:

1. **Strict first.** The input goes through `JSON.parse` before anything else. If it succeeds, nothing is rewritten and the tool says so. The one thing still worth checking on a valid document is duplicate keys, so the tolerant parser is run a second time purely to find those.
2. **Unwrap.** A markdown code fence is located and its body extracted, prose before the JSON is skipped, and the offset of the real payload is recorded so every finding still points at a position in the text you pasted.
3. **Tolerant parse.** A hand-written recursive descent parser walks the text and builds a value directly, recording a finding each time it has to accept something JSON forbids. Repairs happen during parsing rather than by rewriting text, which is why the reported offsets are accurate. When the input ends unexpectedly, open containers are closed and the cut point is recorded.
4. **Verify.** The recovered value is serialized and parsed again with `JSON.parse`. Only output that survives that round trip is shown.

The core is a set of pure functions that take input and return a result object, kept separate from all DOM code, so the parser can be lifted out of the page and run in Node.

**What this tool is guessing about.** Repair is heuristic. Every fix is an inference about what the model meant, not a derivation: single quotes almost certainly meant a string, but a bare word might have meant an enum, and a number with leading zeros might have meant an identifier (this tool keeps it as a string so the zeros are not lost, and says so in the finding). A truncation repair can only close the containers that were open; it cannot invent the content that was never generated, and parseable JSON with silently missing fields is more dangerous than a clean failure. Nothing here validates against your schema, so a repaired document can parse perfectly and still be the wrong shape.

**Structured output modes are strictly better than repairing after the fact.** If you can constrain the response with a tool or function schema, or a JSON-schema output format, the serializer writes the JSON and none of these failure modes can happen. Post-hoc repair is triage for output you do not control, or for a pipeline you have not fixed yet. That is why every finding carries a prevention note: the repair is the short-term fix, the prompt or API change is the real one.

## Privacy

Everything runs in your browser. The text you paste is parsed locally by the JavaScript inside `index.html` and never leaves your machine. There is no server, no analytics, no telemetry, and no external dependencies of any kind, so there is nothing to upload to. You can confirm it by reading the source or by watching the network tab while you use the page.

Model output often contains customer data, internal identifiers, or content under NDA. For anything sensitive, save `index.html` and open it offline. The one thing the page stores is your light or dark theme choice, saved in `localStorage` under the key `llm-json-repair-theme`.

## Run locally

```
git clone https://github.com/0xelitesystem/llm-json-repair
cd llm-json-repair
```

Open `index.html` in any modern browser. Or serve the folder with `python -m http.server` and visit http://localhost:8000.

## Build

No build step. The whole tool is one `index.html` with inline CSS and JavaScript and no dependencies.

## License

MIT. See [LICENSE](LICENSE).

## More

- The full catalog of tools and references: https://0xelitesystem.github.io/
- https://elitesystem.ai
