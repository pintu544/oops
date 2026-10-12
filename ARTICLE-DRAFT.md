# Weekend Agent Challenge: oops — because error messages shouldn't need a translator

## What it does and who it's for

Let me be honest about how `oops` was born. It was late, I was staring at a sqlite traceback for the twentieth time that week, and I caught myself thinking: why do I have to mentally parse this wall of red text every single time? I know what "no such table" means. But after six hours of debugging, my brain just... doesn't.

So `oops` is dead simple. You pipe your error into it, and it gives you three things:

- **What went wrong** — in plain English, one sentence, zero jargon
- **Why it probably happened** — the likely cause, no blame attached
- **The fix** — actual code or a command you can copy-paste

That's the whole product. No config files, no dashboard, no setup wizard.

```bash
npm run build 2>&1 | oops
oops "TypeError: Cannot read properties of undefined (reading 'map')"
cat traceback.txt | oops
```

It's for developers like me — people who'd rather spend their energy fixing the bug than decoding the error message first.

## How I built it

The whole thing is one Python file, about 120 lines, with a single dependency: boto3. I kept the architecture boring on purpose. I had hours, not days.

Here's how it works, step by step:

1. **Input** — argparse grabs your error from stdin, a CLI argument, or a `--file` flag
2. **Trim** — the error is truncated to 6,000 characters so a giant stack trace doesn't blow up the context window
3. **Think** — it's sent to **Amazon Bedrock** (Nova Micro, `us-east-1`) through the converse API. Bedrock is the entire brain — no local models, no regex heuristics, no keyword matching
4. **Answer** — the response lands in under three seconds, shaped by a strict system-prompt contract

Everything interesting lives in that system prompt. It enforces:

- "In plain English" — one sentence, zero jargon
- "Likely cause" — a sentence or two, no condescension
- "The fix" — concrete and copy-pasteable, never vague advice
- Under 200 words total, warm tone throughout

Why Nova Micro? Two reasons, both practical:

- **Cost** — it's one of Bedrock's cheapest models, so each explanation costs a fraction of a cent
- **Speed** — the full round-trip lands under three seconds. That mattered to me because if this took ten seconds I'd never build the habit. It had to feel like a reflex: see red, pipe to oops, keep moving

Auth is just standard AWS environment variables. If you've already got AWS credentials configured, oops just works — nothing else to set up.

The repo is public at https://github.com/pintu544/oops — MIT licensed, `pip install -r requirements.txt` and you're running.

## The delightful detail

Here's the part I care about most: **oops never makes you feel stupid for the error.**

I've tried other error explainers. They're correct, but they're cold — they read like documentation that happens to be about your mistake. And when you're frustrated, when you've been stuck for an hour, the last thing you need is a tool that talks down to you.

So the system prompt carries what I think of as a tone contract:

- It leads with reassurance disguised as clarity
- It doesn't say "you forgot to create the table" — it says "the table doesn't exist yet"
- Same information, completely different feeling. One scolds you, the other helps you

How do I know it worked? I ran it against two real errors:

- A Python `sqlite3.OperationalError` — nailed the cause and gave the exact `CREATE TABLE` fix
- A Node `EADDRINUSE` crash — laid out both options (kill the process, or switch ports) instead of arrogantly choosing for me

Both times it honored the contract: plain-English sentence first, then the cause, then a specific fix with real code.

The moment I knew the tone was right: reading the sqlite output back. "Your code tried to access a table called 'users' that doesn't exist in the database." No blame. No lecture. Just a friend telling you what's up. Honestly, that's the whole product.

## Proof it works

Repo: https://github.com/pintu544/oops

And here's a real run — actual sqlite traceback in, friendly explanation out, in seconds:

[SCREENSHOT]

Clone it, set your AWS credentials, throw any error at it. If it ever talks down to you, open an issue — I'll fix the prompt. That's a promise.

---

*Built for the AWS Builder Center Weekend Challenge (Oct 9–12, 2026). Total build time: about an hour. Cost per explanation: basically nothing.*
