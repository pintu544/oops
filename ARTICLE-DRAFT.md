# Weekend Agent Challenge: oops — the error explainer that doesn't make you feel dumb

## What it does and who it's for

`oops` is a tiny CLI agent for developers. You hit a wall of red traceback text, you pipe it into `oops`, and it tells you — in plain English — what actually went wrong, what probably caused it, and the exact fix you can copy-paste. It's for every developer who has ever stared at `sqlite3.OperationalError: no such table: users` at 2 AM and wished someone would just explain it like a patient friend instead of a manual page.

```bash
npm run build 2>&1 | oops
oops "TypeError: Cannot read properties of undefined (reading 'map')"
cat traceback.txt | oops
```

That's the whole interface. No config files, no setup wizard, no dashboard. It reads from stdin, a CLI argument, or a `--file` flag, and answers in under three seconds.

## How I built it

`oops` is a single Python file (~120 lines) with one dependency: `boto3`. The architecture is deliberately boring:

1. **Input** — argparse + stdin handling grabs the error text, truncates it to 6,000 characters so a giant stack trace doesn't blow up the context window.
2. **Brain** — the error is sent to **Amazon Bedrock** in `us-east-1`, specifically the `us.amazon.nova-micro-v1:0` model, via the `converse` API. Bedrock is the only AWS service in the stack, and it's the entire intelligence layer — no local models, no heuristics.
3. **Personality** — everything interesting lives in the system prompt. It forces a strict output contract: a section called "In plain English" (one sentence, zero jargon), "Likely cause" (1–2 sentences), and "The fix" (a concrete, copy-pasteable code change or command). Total budget: under 200 words, warm tone, zero condescension.

I picked Nova Micro for two practical reasons: it's one of Bedrock's cheapest models, so each explanation costs a fraction of a cent, and its latency keeps the whole round-trip under three seconds — fast enough that piping a failing build into `oops` feels like a reflex, not a detour. The tool authenticates with standard AWS credentials (`AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY`) via environment variables, so it slots into any developer's existing AWS setup with no extra plumbing.

The repo is public at https://github.com/pintu544/oops — MIT licensed, `pip install -r requirements.txt` and you're running.

## The delightful detail

The delightful detail is the tone contract in the system prompt: **oops never makes you feel dumb for the error.**

Every error explainer I've used reads like documentation — correct, but cold. When you're frustrated at midnight, cold correctness isn't what you need. So I wrote the system prompt to behave like the senior dev sitting next to you: it leads with reassurance disguised as clarity ("Your code tried to access a table called 'users' that doesn't exist"), names the cause without blame, and hands you the fix like it's no big deal.

How do I know it worked? I tested it on two real errors — a Python `sqlite3.OperationalError` and a Node `EADDRINUSE` crash. In both cases the output followed the contract exactly: one plain-English sentence first, then cause, then a specific fix with real code. The EADDRINUSE case even offered both options (kill the process or change the port) instead of picking one arrogantly. The moment I knew the tone was right: the output for the sqlite error didn't say "you forgot to create the table" — it said "the table doesn't exist yet," which is the difference between being scolded and being helped. Small word, big feeling.

## Proof it works

Live repo: https://github.com/pintu544/oops

Screenshot below shows a real run — a sqlite traceback piped into `oops`, answered in seconds with the plain-English explanation, likely cause, and fix:

[SCREENSHOT]

Try it yourself: clone the repo, set your AWS credentials, and pipe any error into it. If it doesn't explain your error kindly and correctly, open an issue — I'll fix the prompt.

---

*Built for the AWS Builder Center Weekend Challenge (Oct 9–12, 2026). Total build time: under an hour. Total cost per explanation: a fraction of a cent.*
