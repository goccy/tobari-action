# tobari-action

GitHub Action for [tobari](https://github.com/goccy/tobari) - a scoped coverage measurement tool for Go.

## Actions

### Setup (`setup/`)

Installs the tobari CLI and configures `GOFLAGS` for coverage-instrumented builds.

#### Inputs

| Name | Description | Default |
|------|-------------|---------|
| `version` | Version of tobari to install (e.g., `v0.9.0` or `latest`) | `latest` |
| `embed-code` | Embed source code into instrumented binaries | `false` |

#### Usage

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.24'
      - uses: goccy/tobari-action/setup@v1
      - run: go build ./...
        # GOFLAGS is automatically set, so go build runs with coverage instrumentation
```

With embed-code enabled:

```yaml
      - uses: goccy/tobari-action/setup@v1
        with:
          embed-code: true
      - run: go build -o myapp .
        # The built binary will have source code embedded
```

### Report (`report/`)

Generates an HTML coverage report from `tobari.json` and uploads it as a directly viewable artifact.

#### Prerequisites

The `setup` action must be run before this action to install the tobari CLI.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `tobari-json` | Path to tobari.json file(s). Multiple paths can be specified with newlines. | No | `tobari/tobari.json` |
| `source` | Path to source.tar.gz file(s) for source resolution. Multiple paths can be specified with newlines. | No | |
| `binary` | Path to binary with embedded source code | No | |
| `name` | Artifact name for the HTML report | No | `tobari-report` |

> **Note:** `source` and `binary` are mutually exclusive. Specify at most one.

#### Usage

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: '1.24'
      - uses: goccy/tobari-action/setup@v1
      - run: go test ./...
      - uses: goccy/tobari-action/report@v1
        # Reads tobari/tobari.json by default
```

With source archive:

```yaml
      - uses: goccy/tobari-action/report@v1
        with:
          source: sources.tar.gz
```

With embedded source binary:

```yaml
      - uses: goccy/tobari-action/setup@v1
        with:
          embed-code: true
      - run: go build -o myapp .
      - run: go test ./...
      - uses: goccy/tobari-action/report@v1
        with:
          binary: myapp
```

With multiple tobari.json files:

```yaml
      - uses: goccy/tobari-action/report@v1
        with:
          tobari-json: |
            service-a/tobari/tobari.json
            service-b/tobari/tobari.json
```

## License

MIT
