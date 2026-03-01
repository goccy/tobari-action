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

## License

MIT
