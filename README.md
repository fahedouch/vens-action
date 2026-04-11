# vens-action

GitHub Action for [vens](https://github.com/venslabs/vens) — context-aware vulnerability prioritization with OWASP risk scoring and CycloneDX VEX output.

Drop this action into your pipeline after a Trivy or Grype scan step to get a contextual VEX file (and optionally an enriched report) — without installing Go or managing binaries yourself.

## Quick start

```yaml
- name: Scan image with Trivy
  run: trivy image python:3.11-slim --format json --output report.json

- name: Prioritize with vens
  uses: venslabs/vens-action@v0.3.1
  with:
    version: v0.3.1
    config-file: .vens/config.yaml
    input-report: report.json
    llm-provider: openai
    llm-model: gpt-4o
    llm-api-key: ${{ secrets.OPENAI_API_KEY }}
    enrich: "true"

- uses: actions/upload-artifact@v4
  with:
    name: vens-output
    path: |
      vens-vex.cdx.json
      vens-enriched.json
```

## Inputs

| Name | Required | Default | Description |
|---|---|---|---|
| `version` | yes | — | vens release tag to install (e.g. `v0.3.1`). `latest` is not yet supported. |
| `config-file` | yes | — | Path to `config.yaml` describing project context (exposure, data sensitivity, etc.). |
| `input-report` | yes | — | Path to the Trivy or Grype JSON scan report. |
| `input-format` | no | `auto` | Scanner format: `auto` \| `trivy` \| `grype`. |
| `output-vex` | no | `vens-vex.cdx.json` | Output path for the generated CycloneDX VEX. |
| `sbom-serial-number` | no | *(auto)* | BOM-Link serial (`urn:uuid:...`). Auto-generated if empty. |
| `sbom-version` | no | `1` | BOM-Link version integer. |
| `llm-provider` | no | `openai` | `openai` \| `anthropic` \| `googleai` \| `ollama` \| `mock`. |
| `llm-model` | no | *(provider default)* | Model name (mapped to `<PROVIDER>_MODEL`). |
| `llm-api-key` | no | — | API key for the selected provider. Masked in logs. |
| `llm-base-url` | no | — | Custom base URL (OpenAI-compatible endpoints or Ollama host). |
| `llm-batch-size` | no | `10` | CVEs per LLM request. |
| `enrich` | no | `false` | If `true`, also run `vens enrich` to produce an enriched Trivy report. |
| `output-enriched-report` | no | `vens-enriched.json` | Output path for the enriched report. |
| `debug-dir` | no | — | Dump LLM prompts and responses to this directory. |
| `working-directory` | no | `.` | Working directory for the run. |

## Outputs

| Name | Description |
|---|---|
| `vex-file` | Path to the generated CycloneDX VEX file. |
| `enriched-report` | Path to the enriched Trivy report (empty when `enrich=false`). |
| `sbom-serial-number` | SBOM serial number that was used (useful when auto-generated). |
| `version` | vens version that was installed. |

## LLM provider mapping

The action maps `llm-api-key` and `llm-model` to the correct environment variables based on `llm-provider`:

| `llm-provider` | API key env | Model env | Base URL env |
|---|---|---|---|
| `openai` | `OPENAI_API_KEY` | `OPENAI_MODEL` | `OPENAI_BASE_URL` |
| `anthropic` | `ANTHROPIC_API_KEY` | `ANTHROPIC_MODEL` | — |
| `googleai` | `GOOGLE_API_KEY` | `GOOGLE_MODEL` | — |
| `ollama` | — | `OLLAMA_MODEL` | `OLLAMA_HOST` |
| `mock` | — | — | — |

The API key is masked via `::add-mask::` before any other step runs.

## Supported runners

- `ubuntu-*` (x64, arm64)
- `macos-*` (x64, arm64)

Windows runners are not yet supported — the action fails fast with a clear error.

## Versioning

Pin the action to a vens release:

```yaml
uses: venslabs/vens-action@v0.3.1
```

## License

Apache License 2.0 — see [LICENSE](LICENSE).
