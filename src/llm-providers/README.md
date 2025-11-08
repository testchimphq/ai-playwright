# LLM Providers

The modules in this folder implement the pluggable LLM provider architecture used by `ai-wright`.

## Built-in Providers

The library ships with the following providers out of the box (checked in this order):

- `openai` (`OPENAI_API_KEY`, optional `OPENAI_MODEL`)
- `testchimp-key` (`TESTCHIMP_API_KEY` + `TESTCHIMP_PROJECT_ID`)
- `testchimp-pat` (`TESTCHIMP_USER_AUTH_KEY` + `TESTCHIMP_USER_MAIL`)
- `gemini` (`GEMINI_API_KEY`, optional `GEMINI_MODEL`)
- `claude` (`CLAUDE_API_KEY`, optional `CLAUDE_MODEL`)

## Provider Responsibilities

Each provider implements the `LLMProvider` interface defined in `llm-provider.ts`. A provider must:

- expose a human-readable `name` (e.g. `openai`, `testchimp-key`);
- implement `canAuthenticate()` to detect whether required environment variables are available; and
- implement `callLLM()` to send the request and return the raw JSON string reply from the underlying LLM.

The shared `LLMRequest` includes the `systemPrompt`, `userPrompt`, and an optional base64-encoded screenshot (`image`). Providers receive a `timeoutMs` value via `LLMCallOptions` and should pass it to their underlying HTTP or SDK call.

## Adding a New Provider

1. Create a new file in this folder that exports a factory function returning an object that satisfies `LLMProvider`.
2. Detect authentication inside `canAuthenticate()` by inspecting environment variables. Do not throw from this method—return `false` if credentials are missing.
3. Implement `callLLM()` to invoke the target model and return a JSON string. Throw when the HTTP call fails or the response cannot be parsed.
4. If multiple providers share behavior (e.g. identical endpoints), factor the common code into a helper module (see `testchimp-common.ts` for an example).
5. Register the new provider by adding it to `LLM_PROVIDER_ORDER` in `config.ts` in the desired priority order. Providers are checked sequentially until one reports that it can authenticate.

With these steps complete, the library will automatically select the new provider when its credentials are present.
