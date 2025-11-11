# landscape-openapi-spec

[![Bundle and sync API Client with OpenAPI spec, publish GitHub release/Go package](https://github.com/jansdhillon/landscape-openapi-spec/actions/workflows/sync-client.yaml/badge.svg)](https://github.com/jansdhillon/landscape-openapi-spec/actions/workflows/sync-client.yaml)

OpenAPI 3.1 specification for the Landscape Server API.

## Validating, linting, and bundling

Make sure you have `swagger-cli` and `spectral-cli` installed:

```sh
npm install -g @apidevtools/swagger-cli @stoplight/spectral-cli
```

Then, you can use the `make` recipes to validate, lint, and bundle the OpenAPI spec.

Validate:

```sh
make validate
```

Lint:

```sh
make lint
```

Generate bundle in `openapi/landscape_api.bundle.yaml`:

```sh
make bundle
```

Example usage: <https://github.com/jansdhillon/landscape-go-api-client>.

## Syncing

Whenever changes are pushed to `main`, [a GitHub Actions workflow](./.github/workflows/sync-client.yaml) is triggered that bundles the OpenAPI spec, creates a release for this repository and the Go client, and uses the new bundle to open a PR on `landscape-go-api-client` to update the generated code. The workflow also syncs the Go package with [pkg.go.dev](https://pkg.go.dev/github.com/jansdhillon/landscape-go-api-client).
