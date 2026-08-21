---
name: codex-model-provider
description: Configure Codex to use a model provider or switch models by editing config.toml and the model catalog safely. Use when a user asks to add an API provider, change the active model, or expose a provider-supported model in Codex.
---

# Configure a Codex model provider

Use this skill when the user wants Codex to call a different model provider or model.

## Required decisions

1. Identify the provider's exact model ID.
2. Identify the provider's base URL.
3. Confirm the request protocol (`responses`, `chat`, or another protocol supported by the installed Codex version).
4. Decide whether the model should become the default or only be added to the model catalog.
5. Keep credentials out of files whenever an environment-variable reference is supported.

## Configuration workflow

1. Read the current `config.toml` and preserve unrelated settings.
2. Add or update a `[model_providers.<provider>]` section.
3. Set the top-level `model_provider` to the provider when the user wants it active.
4. Set the top-level `model` only when the user wants to change the default.
5. If the model should appear in the picker, add a complete entry to the file referenced by `model_catalog_json`.
6. Keep providers with different protocols separate. Do not use a single `responses` provider for a model documented as Chat Completions or Messages.
7. Restart Codex after editing so the application reloads the configuration.

## Credential handling

Prefer:

```toml
env_key = "PROVIDER_API_KEY"
```

and ask the user to set `PROVIDER_API_KEY` in the operating-system environment. Never print, copy, or commit the real secret. If the current configuration already contains an inline token, avoid exposing it and recommend rotating it if it was shared.

## Model catalog guidance

The model catalog controls discovery and model metadata; it does not grant API access. A catalog entry must use the provider's exact model ID. Do not claim that a model works solely because it appears in the picker. Validate the provider's protocol and permissions separately.

When adding an entry, preserve the surrounding schema and provide at least:

- `slug`
- `display_name`
- `description`
- `visibility`
- `supported_in_api`
- `default_reasoning_level`
- `supported_reasoning_levels`

Copying an existing entry is acceptable only when the copied capabilities are actually valid for the new model.

## Verification

After editing:

- Parse the JSON model catalog.
- Check that `model_provider` names an existing provider section.
- Check that `model` matches a catalog `slug` when a custom catalog is in use.
- Confirm that the selected provider's `wire_api` matches the model's documented protocol.
- Tell the user which files changed and whether the default model changed.

