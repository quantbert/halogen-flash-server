# Use this server from OMP

This server exposes an OpenAI-compatible API that OMP can use as a custom
provider.

## Required first step: ask for the host

Before changing any OMP configuration, **ask the user for the host address that
the OMP machine should use to reach the server**. Do not assume
`127.0.0.1`, `localhost`, `aiw11.hi.fin`, or any other hostname.

Ask:

> What host address should this OMP installation use to reach the Halogen
> server?

The answer may be a hostname, an IP address, or a complete URL. Build the API
base URL as follows:

- Hostname or IP: `http://<HOST_ADDRESS>:8731/v1`
- Complete URL: preserve its scheme and port, and ensure it ends in `/v1`

Examples only—never select one without asking the user:

- `127.0.0.1` becomes `http://127.0.0.1:8731/v1`
- `server.example.internal` becomes `http://server.example.internal:8731/v1`
- `https://llm.example.com` becomes `https://llm.example.com/v1`

## Verify the address

Before editing OMP configuration, verify that the supplied address reaches a
Halogen server:

```bash
curl --fail --silent http://<HOST_ADDRESS>:8731/health
curl --fail --silent http://<HOST_ADDRESS>:8731/v1/models
```

Adjust the scheme and port when the user supplied a complete URL. Continue only
if `/health` reports `"status":"ok"` and `/v1/models` lists
`halogen-qwen3.8-flash-next`.

## Configure the provider

Merge the following provider into `~/.omp/agent/models.yml`. Preserve every
existing provider. Replace `<BASE_URL>` with the verified URL ending in `/v1`.

```yaml
providers:
  halogen:
    baseUrl: <BASE_URL>
    auth: none
    api: openai-responses
    models:
      - id: halogen-qwen3.8-flash-next
        name: Halogen Qwen3.8 Flash Next
        reasoning: true
        input: [text]
        tokenizer: qwen3
        contextWindow: 262144
        maxTokens: 65536
        supportsTools: true
        cost:
          input: 0
          output: 0
          cacheRead: 0
          cacheWrite: 0
```

If `models.yml` does not exist, create it with the content above after replacing
`<BASE_URL>`.

`auth: none` is intentional: Halogen does not authenticate API clients. Use a
trusted private network, VPN, or authenticated reverse proxy. Do not expose the
raw endpoint to an untrusted network.

## Test OMP

Start a new OMP process and make a real request through the configured provider:

```bash
omp --no-session --model halogen/halogen-qwen3.8-flash-next \
  --thinking off -p 'Reply with exactly: HALOGEN_OK'
```

The expected response is:

```text
HALOGEN_OK
```

## Selecting the model

Use the local server for one session:

```bash
omp --model halogen/halogen-qwen3.8-flash-next
```

Only make it OMP's default when the user explicitly requests that change. If
requested, merge this setting into `~/.omp/agent/config.yml` while preserving
all unrelated configuration:

```yaml
modelRoles:
  default: halogen/halogen-qwen3.8-flash-next:medium
```

A configured provider does not by itself change OMP's default model.
