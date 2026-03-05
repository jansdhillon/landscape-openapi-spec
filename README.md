# landscape-openapi-spec

[![Bundle and publish GitHub release](https://github.com/jansdhillon/landscape-openapi-spec/actions/workflows/release.yaml/badge.svg)](https://github.com/jansdhillon/landscape-openapi-spec/actions/workflows/release.yaml)

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

View docs:

```sh
make serve-docs
```

Example usage: <https://github.com/jansdhillon/landscape-go-api-client>.

## Releasing

Whenever a new version of the OpenAPI spec is pushed to `main`, [a GitHub Actions workflow](./.github/workflows/release.yaml) bundles the spec and publishes a GitHub release with the bundle as an artifact.

The workflow can also be triggered externally via `repository_dispatch` (event type `openapi-released`), allowing upstream repositories to kick off a release without a direct push to `main`. The sending workflow must use a PAT with `repo` scope on this repository, as `GITHUB_TOKEN` is scoped to the originating repo only.
