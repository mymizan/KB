# OpenClaw + OpenRouter: "Unknown model" Error

## Symptoms

After selecting the **OpenRouter** provider and choosing a model from
the TUI (for example, `google/gemini-2.5-flash`), OpenClaw fails when
sending a request.

Example error:

``` text
FailoverError: Unknown model: google/gemini-2.5-flash
Found agents.defaults.models["google/gemini-2.5-flash"]
```

or

``` text
Unknown model
```

------------------------------------------------------------------------

## Cause

This appears to be a mismatch between the model selected in the OpenClaw
TUI and the models defined in the configuration file.

Selecting a model from the TUI does **not** always update the
configuration correctly. As a result, the gateway cannot resolve the
model at runtime, even though it was selected successfully.

------------------------------------------------------------------------

## Resolution

1.  Open the OpenClaw configuration file.
2.  Verify that the provider is configured correctly:

``` yaml
provider: openrouter
```

3.  Explicitly list every model that should be available under the
    OpenRouter provider.

Example:

``` yaml
models:
  - google/gemini-2.5-flash
  - anthropic/claude-sonnet-4
  - openai/gpt-5
```

4.  Save the configuration.
5.  Restart the container:

``` bash
docker compose down
docker compose up -d
```

6.  Test again.

------------------------------------------------------------------------

## Notes

-   Do not assume that selecting a model in the TUI updates the
    configuration.
-   If you receive an **Unknown model** error, check the configuration
    file first.
-   Restart the container after changing the configuration.
-   If the problem persists, compare the model identifier in the
    configuration with the exact identifier expected by the OpenRouter
    provider.

------------------------------------------------------------------------

## Takeaway

If OpenClaw reports **Unknown model** while using the OpenRouter
provider, the first thing to verify is the configuration file---not the
API key or the selected provider. In my case, the TUI selection alone
was not sufficient.
