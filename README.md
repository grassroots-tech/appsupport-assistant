# appsupport-assistant

An AI-assisted **Application Support** tool for the Grassroots Tech Movement (GTM) platform.

Support engineers ask a plain-English question ("pod in CrashLoopBackOff, what do I do?"). The tool searches the project's [GitHub wiki](https://github.com/grassroots-tech/appsupport-assistant/wiki) and answers **only from the documented runbooks, with a link to the source page**. If the docs don't cover the question, it says so and tells you to escalate. It never guesses.

## How it works

```
wiki pages (.md) ──► split by heading ──► keyword search (ranked) ──► top 3 sections
                                                                         │
                                         default: print the best section + sources
                                         --ai:    Amazon Bedrock writes a short answer
                                                  grounded ONLY in those sections
```

| File | Purpose |
|---|---|
| `wiki_loader.py` | Reads the cloned wiki and splits each page into sections |
| `search.py` | Ranks sections by keyword match (rarer words count more) |
| `bedrock_client.py` | Sends the question + top sections to Amazon Bedrock (Converse API) |
| `ask.py` | Command-line entry point |

## Quick start

```bash
git clone https://github.com/grassroots-tech/appsupport-assistant.git
cd appsupport-assistant
git clone https://github.com/grassroots-tech/appsupport-assistant.wiki.git wiki
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt

python3 ask.py "site is down"            # keyword search, free
python3 ask.py "site is down" --ai       # Bedrock-written answer (needs AWS credentials)
```

## Design choices

- **Grounded answers only:** the model sees only the wiki excerpts and must reply "Not in the docs - escalate" when they don't answer the question.
- **Works without AI:** keyword mode costs nothing and needs no cloud account.
- **Least-privilege AWS:** a dedicated IAM user that can only call `bedrock:InvokeModel`.
- **Clear exit codes:** `0` answered, `1` error, `2` not in the docs. Easy to wire into scripts or alerts.

## Status

Wiki: live. Code: in progress.
