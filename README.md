# oops

Paste an error, get a plain-English explanation and a fix. A tiny Bedrock-powered CLI agent built for the AWS Builder Center Weekend Challenge.

## What it does

You hit a wall of red text. You pipe it into `oops`. It tells you — in one friendly sentence — what actually went wrong, the likely cause, and a concrete copy-pasteable fix. No jargon dumps, no lectures.

```bash
npm run build 2>&1 | oops
```

```bash
oops "TypeError: Cannot read properties of undefined (reading 'map')"
```

```bash
cat traceback.txt | oops
oops --file error.log
```

## How it works

`oops` sends your error to **Amazon Bedrock** (`us.amazon.nova-micro-v1:0` in `us-east-1`) with a carefully written system prompt that forces three things: a plain-English one-liner, a likely cause, and a specific fix — all under 200 words. The delightful detail: it never makes you feel dumb for the error. It reads like a patient friend, not documentation.

## Setup

```bash
pip install -r requirements.txt
export AWS_ACCESS_KEY_ID=...
export AWS_SECRET_ACCESS_KEY=...
export AWS_DEFAULT_REGION=us-east-1
chmod +x oops.py
```

Tip: alias it for instant access — `alias oops='python3 ~/oops/oops.py'`.

## Cost

Nova Micro is one of Bedrock's cheapest models — each explanation costs a fraction of a cent.

## License

MIT
