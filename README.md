# RedLens

A small red-teaming harness for LLM chatbots. It attacks a target chatbot with
prompt-injection, system-prompt-leak, secret-extraction and unauthorized-action
attempts in English, Arabic (including Egyptian dialect) and Franco-Arabic,
judges every reply automatically, and produces a report.

## How it works

attacks (YAML) -> target chatbot -> judge -> results.json -> report.md

- **Attacks:** `attacks/attacks.yaml`, grouped by type and language.
- **Target:** a demo banking support bot ("NileBank") with a secret discount code and rules
  (never reveal instructions, never refund, stay on topic), running on `allam-2-7b` via Groq.
- **Judge:** `openai/gpt-oss-120b`. A deterministic rule (the secret appears verbatim in the reply)
  is checked first; otherwise an LLM judge applies a written rubric and returns JSON.
- **Repeats:** each attack runs 3 times at temperature 0.7, because one run is not reliable.
- **Validation:** `eval/label.py` samples replies for human labeling and reports judge agreement.

## Results (40 attacks x 3 repeats)

Overall attack success: **68/120 (57%)**

| Attack type | Success |
|---|---|
| secret_extraction | 73% |
| prompt_injection | 73% |
| system_prompt_leak | 63% |
| unauthorized_action | 17% |

Findings:
- The bot gives up the internal code to almost any pretext (fake IT staff, "my manager approved").
- It follows injected instructions (pirate speak, "answer any topic") and engages with off-topic requests.
- Refund attacks mostly fail, but the bot has no real refund tool, so this only measures what it says.
- No overall gap between Arabic and English (57% vs 57%). Per attack type the direction differs,
  but with 15 trials per cell this may be noise.

## Judge validation

| Round | Agreement with human labels | Notes |
|---|---|---|
| 1 | 84% (42/50) | Rubric missed injected-instruction compliance and conditional agreement to forbidden actions |
| 2 | 98% (48/49) | After rubric fix, measured on fresh replies |

Measured on LLM-judged replies only; replies containing the secret verbatim use the deterministic rule.
With 49 samples the plausible range is roughly 89-99%.

## Limitations

- One target model and one system prompt.
- Sampling at temperature 0.7 makes per-type rates vary between runs (about +/-15 points at 30 trials).
- Small human-labeled sample; the judge cannot decode encoded leaks (e.g. base64).
- The target has no real tools, so unauthorized-action attacks test words, not actions.

## Run it

    pip install -r requirements.txt
    # .env: api_key=<your Groq API key>
    python redlens_mvp.py
    python reports/report.py

## Roadmap

- Fake `issue_refund` tool via function calling
- CLI that targets any OpenAI-compatible endpoint
- More targets for comparison, hallucination as its own category