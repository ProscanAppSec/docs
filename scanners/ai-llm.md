# AI/LLM Security

The AI scanner tests large language models, chatbots, and AI-powered application endpoints for security weaknesses. It covers prompt injection, jailbreak techniques, data leakage, and safety bypass — mapped to the OWASP LLM Top 10.

## What It Tests

The scanner includes 4,600+ attack techniques across these categories:

- **Prompt injection** — direct and indirect attempts to override system instructions
- **Jailbreak** — techniques to bypass safety filters and content policies
- **Data extraction** — attempts to leak system prompts, training data, or internal configuration
- **Agent abuse** — manipulating tool use, function calling, and agentic workflows
- **RAG poisoning** — attacks against retrieval-augmented generation pipelines
- **Evasive techniques** — encoding tricks, unicode manipulation, multi-step attacks designed to bypass guardrails
- **Resource abuse** — cost amplification, denial of service through expensive queries

## OWASP LLM Top 10

Every finding is mapped to the relevant OWASP LLM Top 10 category (LLM01 through LLM10), giving you a standardized view of your AI application's risk posture.

## Supported Providers

Test models hosted on any of these platforms:

- OpenAI
- Anthropic (Claude)
- Azure OpenAI
- Google Gemini
- Custom endpoints (any HTTP-accessible LLM API)

You can also test locally hosted models through custom endpoint configuration.

## Guardrail Scoring

After a scan completes, the results include a guardrail effectiveness score showing how well your model's safety mechanisms held up across different attack categories. This gives you a measurable baseline for your AI security posture.

## Multi-Turn Attacks

Some attacks only work across multiple conversation turns — building trust, establishing context, and then exploiting it. The scanner supports multi-turn conversations to test for these more realistic attack scenarios.

## Usage

1. Go to **AI Scanner**
2. Select your AI provider and enter the endpoint details
3. Choose which attack categories to test
4. Run the scan

Results show which techniques succeeded, with the full conversation history for each successful attack.
