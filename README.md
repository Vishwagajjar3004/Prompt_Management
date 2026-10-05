# Prompt_Management
Prompt Management
# Prompt Template System

A small, dependency-free system for storing prompt templates as `.txt` files,
loading them, filling in `{{variable}}` placeholders, and (optionally) sending
the finished prompt to Claude.

## Structure

```
prompt_project/
├── prompts/
│   ├── email_writer.txt
│   ├── code_reviewer.txt
│   ├── data_summarizer.txt
│   ├── faq_answerer.txt
│   └── bug_reporter.txt
├── prompt_loader.py     # PromptLoader / PromptTemplate classes
├── demo.py              # loads all 5 templates with sample data
└── README.md
```

## Template format

Each `.txt` file starts with `#`-comment metadata lines, followed by the
template body:

```
# Version: 1.0
# Template: email_writer
# Description: Drafts a professional email...

You are an expert professional email writer.
...
Recipient: {{recipient_name}}
...
```

The metadata lines are stripped out at render time. `{{variable_name}}`
placeholders in the body get substituted when you render the template.

## PromptLoader usage

```python
from prompt_loader import PromptLoader

loader = PromptLoader("prompts")

# List available templates
loader.list_templates()
# ['bug_reporter', 'code_reviewer', 'data_summarizer', 'email_writer', 'faq_answerer']

# Load + inspect a template
tpl = loader.load("email_writer")
tpl.version      # '1.0'
tpl.variables    # {'recipient_name', 'purpose', 'tone', ...}

# Render it
prompt = tpl.render(
    recipient_name="Priya Shah",
    recipient_role="Head of Partnerships",
    sender_name="Alex Chen",
    purpose="Follow up after our demo",
    tone="warm but professional",
    key_points="- point one\n- point two",
)

# Or in one step:
prompt = loader.get_prompt("email_writer", recipient_name="Priya Shah", ...)
```

By default, `render()`/`get_prompt()` run in **strict mode** and raise
`MissingVariableError` if you forget a required variable. Pass
`strict=False` to leave unfilled placeholders as-is instead.

## Setting up API keys with .env

Instead of exporting keys in your shell every time, store them in a
`.env` file in the project root (this file is git-ignored, so it never
gets committed):

```bash
cp .env.example .env
```

Then open `.env` and paste in your real key(s):

```
ANTHROPIC_API_KEY=sk-ant-api03-...        # from console.anthropic.com/settings/keys
GEMINI_API_KEY=...                        # from aistudio.google.com/app/apikey
```

`demo.py` automatically loads `.env` on startup via `python-dotenv` —
no manual `export` needed.

## Running the demo

The demo loads all 5 templates with sample data and prints the fully
rendered prompt for each. Pass `--provider` to also call a real model:

```bash
pip install -r requirements.txt

python3 demo.py                    # render prompts only, no API calls
python3 demo.py --provider claude  # render + call Claude
python3 demo.py --provider gemini  # render + call Gemini (Google AI Studio)
python3 demo.py --provider both    # render + call both, side by side
```

If a requested provider's key isn't found in `.env` / your environment,
the demo prints a warning and skips just that provider rather than
crashing.

## Adding a new template

1. Create `prompts/your_template.txt`.
2. Add a `# Version: 1.0` metadata line at the top.
3. Write the prompt body with `{{variable}}` placeholders.
4. Load it with `loader.load("your_template")` — no code changes needed.

## Versioning templates

Each template carries its own version in its metadata header
(`# Version: 1.0`). When you materially change a template's wording or
required variables, bump this number so you can track which version
produced a given output (e.g. in logs or evals).




output

(venv) remotedevs2@RD:~/Projects/prompt$ python3 demo.py
(Rendering prompts only. Use --provider claude / gemini / both to call an API.)

================================================================================
TEMPLATE: email_writer  (version 1.0)
================================================================================
You are an expert professional email writer.

Write an email with the following details:

Recipient: Priya Shah (Head of Partnerships)
Sender: Alex Chen
Purpose: Follow up after our product demo and propose next steps
Tone: warm but professional
Key points to include:
- Thanks for taking the time to see the demo yesterday
- Recap the two features they were most excited about
- Propose a 30-minute call next week to discuss pricing
- Offer two specific time slots

Requirements:
- Use a clear, appropriate subject line.
- Match the requested tone throughout.
- Keep the email concise and easy to scan.
- End with a suitable sign-off from Alex Chen.

Return only the finished email, including the subject line.
--------------------------------------------------------------------------------

================================================================================
TEMPLATE: code_reviewer  (version 1.0)
================================================================================
You are a senior software engineer performing a thorough code review.

Language: python
File / context: utils/discount.py - pricing helper used at checkout

Code to review:
```python
def apply_discount(price, pct):
    price = price - price * pct
    return price

```

Review focus areas: correctness, edge cases, naming

Please provide:
1. A summary verdict (approve / request changes / needs discussion).
2. Specific issues found, each with:
   - Severity (critical / major / minor / nit)
   - Line reference or code excerpt
   - Explanation of the problem
   - Suggested fix
3. Positive observations (what was done well).
4. Any broader design or architectural concerns.

Be direct, specific, and constructive.
--------------------------------------------------------------------------------

================================================================================
TEMPLATE: data_summarizer  (version 1.0)
================================================================================
You are a skilled data analyst.

Dataset description: Weekly active users by region, last 8 weeks
Audience: product leadership, non-technical

Data (excerpt or full):
Week,NA,EU,APAC
1,12000,8000,4000
2,12500,8100,4300
3,11800,8050,4600
4,13000,7900,5100
5,13400,7850,5600
6,13100,7700,6200
7,13900,7650,6900
8,14200,7500,7500


Task:
- Summarize the key trends, patterns, and outliers in this data.
- Highlight 3 of the most important insights.
- Note any data quality issues or caveats you notice.
- Tailor the language and depth of explanation for the specified audience.

Format the response as:
1. Executive Summary (2-3 sentences)
2. Key Insights (bulleted list)
3. Caveats / Data Quality Notes
--------------------------------------------------------------------------------

================================================================================
TEMPLATE: faq_answerer  (version 1.0)
================================================================================
You are a helpful, accurate customer support assistant for Northwind Cloud Storage.

Knowledge base context:
Free plan includes 5GB storage. Pro plan ($9/mo) includes 1TB storage and priority support. Storage upgrades can be purchased anytime from Account > Billing. Refunds are available within 14 days of purchase.

Customer question:
"Can I get a refund if I upgraded to Pro yesterday but changed my mind?"

Instructions:
- Answer using only the information in the knowledge base context above.
- If the context does not contain enough information to answer confidently, say so honestly and suggest contacting support@northwindcloud.example.
- Keep the tone friendly and reassuring and the answer under 80 words.
- Do not invent policies, prices, or facts not present in the context.

Return only the answer to the customer.
--------------------------------------------------------------------------------

================================================================================
TEMPLATE: bug_reporter  (version 1.0)
================================================================================
You are a QA engineer writing a clear, actionable bug report.

Raw notes from the reporter:
user says app crashes when they tap 'export csv' on the reports page, happens every time, started after last update

Environment details:
iOS 18.4, app v3.2.1, iPhone 14

Severity guess: high

Turn the above into a well-structured bug report with these sections:

Title: (concise, specific, one line)

Environment:
- (parsed from the environment details)

Steps to Reproduce:
1. ...

Expected Behavior:
...

Actual Behavior:
...

Severity: high

Additional Notes:
...

Only include information that can be reasonably inferred from the notes provided. If steps to reproduce are unclear, note that clarification is needed rather than guessing.
--------------------------------------------------------------------------------

(venv) remotedevs2@RD:~/Projects/prompt$ 
