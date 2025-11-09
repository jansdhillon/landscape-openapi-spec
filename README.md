# landscape-openapi-spec

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

Bundle (overwrite `landscape_api.bundle.yaml`:

```sh
make bundle
```

Example usage: https://github.com/jansdhillon/landscape-go-api-client


