# landscape-openapi-spec

## Project overview

This repository is the source-of-truth OpenAPI 3.1 specification for the [Canonical Landscape](https://ubuntu.com/landscape) Server REST API. It is authored as a multi-file spec (split across `openapi/components/`) and bundled into a single `landscape_api.bundle.yaml` artifact on every merge to `main`.

The bundle drives downstream code generation: when a new version is published, a GitHub Actions workflow regenerates the [landscape-go-api-client](https://github.com/jansdhillon/landscape-go-api-client) and opens a PR there with the updated generated code.

---

## Repository structure

```
openapi/
  openapi.yaml                    # Root spec entry point — paths, tags,
  │                               # component refs, version number
  landscape_api.bundle.yaml       # Generated bundle (do not edit manually)
  components/
    paths/
      script.yaml                 # Path item definitions for /api/scripts/…
      actions.yaml                # Legacy /api path
      auth.yaml                   # /api/login paths
    schemas/
      script.yaml                 # All script and script-profile schemas
      auth.yaml                   # Auth-related schemas
      error.yaml                  # Error schema
      legacy.yaml                 # Legacy action schemas
    parameters/
      script.yaml                 # Path/query params for scripts & script profiles
      legacy.yaml                 # Legacy params
    responses/
      script.yaml                 # Response definitions for scripts & script profiles
      error.yaml                  # Reusable error responses
      legacy.yaml                 # Legacy responses
.github/
  workflows/
    lint.yaml                     # Runs on PRs: make validate && make lint
    sync-client.yaml              # Runs on push to main: bundle → release → regen client
.spectral.yaml                    # Spectral lint ruleset (extends oas, all)
Makefile                          # validate / lint / bundle targets
```

---

## CI/CD pipeline

### On pull request (`lint.yaml`)
1. Installs `swagger-cli` and `@stoplight/spectral-cli`.
2. Runs `make validate` — resolves all `$ref`s and validates the spec is well-formed OpenAPI 3.1.
3. Runs `make lint` — applies the Spectral ruleset; **fails on any warning or higher**.

### On push to `main` (`sync-client.yaml`)
1. Runs `make bundle` → produces `openapi/landscape_api.bundle.yaml`.
2. Reads `info.version` from the bundle.
3. Tags this repo as `v{version}` and creates/updates a GitHub release with the bundle as an artifact.
4. Downloads the bundle and regenerates the Go API client, then opens a PR on `landscape-go-api-client`.

---

## How to add or change API endpoints

### 1. Check the source of truth

Routes live in `canonical/landscape/api/urls.py` in [canonical/landscape-server](https://github.com/canonical/landscape-server). Check URL patterns, HTTP methods, and handler signatures there before writing spec.

Handler files (e.g. `canonical/landscape/api/script_profile.py`) contain the Pydantic request/response models — use these to derive schemas.

### 2. Edit component files

Always edit the component files under `openapi/components/`, never `openapi.yaml` directly (except for registration and version bump).

| What you're adding          | File to edit                          |
| --------------------------- | ------------------------------------- |
| New path/operation          | `components/paths/<domain>.yaml`      |
| New schema                  | `components/schemas/<domain>.yaml`    |
| New path or query parameter | `components/parameters/<domain>.yaml` |
| New response                | `components/responses/<domain>.yaml`  |
| New error response          | `components/responses/error.yaml`     |

### 3. Register in `openapi.yaml`

After adding components, register all new items in `openapi/openapi.yaml`:
- Add path entries under `paths:` using JSON Pointer encoding (`~1` for `/`, `:` is literal).
- Add schema refs under `components.schemas:`.
- Add parameter refs under `components.parameters:`.
- Add response refs under `components.responses:`.
- Add any new tags under `tags:` **in alphabetical order** (enforced by Spectral).

### 4. Bump the version — **required to trigger release CI**

The `sync-client.yaml` workflow reads `info.version` from the bundle. Pushing to `main` without a version bump will overwrite the existing release tag and regenerate code unnecessarily (or not at all, if the tag already exists).

Increment the patch version in `openapi/openapi.yaml`:

```yaml
info:
  version: "0.0.9"   # was 0.0.8
```

Use semantic versioning (`MAJOR.MINOR.PATCH`):
- **PATCH**: adding new endpoints, fixing descriptions/schema details.
- **MINOR**: deprecating fields, adding optional request body fields, new tags.
- **MAJOR**: breaking changes (removing endpoints, renaming fields, changing response shapes).

### 5. Validate and lint locally

```sh
make validate   # must pass (zero errors)
make lint       # must pass (zero warnings under -F warn)
make bundle     # regenerate the bundle to verify output
```

### 6. Open a PR targeting `main`

Lint CI runs automatically on the PR. `sync-client.yaml` only fires on merge to `main`.

---

## Spectral lint rules (`.spectral.yaml`)

Extends `spectral:oas, all`. Key enforced rules (warnings = CI failure):

| Rule                          | What it checks                                            |
| ----------------------------- | --------------------------------------------------------- |
| `schema-property-description` | Every property in every schema must have a `description`. |
| `info-description-required`   | `info.description` must be present.                       |
| `openapi-tags-alphabetical`   | `tags` array in `openapi.yaml` must be sorted A–Z.        |
| `oas3-schema`                 | Disabled (off).                                           |
| `oas3-unused-component`       | Disabled (off).                                           |

---

## Conventions

- **SPDX header**: All files start with `# SPDX-License-Identifier: Apache-2.0`.
- **Terraform SDK tags**: All schema objects and properties carry `x-oapi-codegen-extra-tags: tfsdk: "<field_name>"` for Terraform provider code generation.
- **Discriminated unions**: Trigger types and script result types use `oneOf` with a `discriminator.propertyName` + `mapping`. The discriminator property itself must have a `description`.
- **Nullable fields**: Use `nullable: true` (OAS 3.0 style) consistently — the bundler and downstream tooling expect this.
- **Action endpoints**: Custom actions use the `{resource}:action` URL pattern (e.g. `/api/scripts/{id}:archive`), matching Landscape Server's werkzeug routing convention.
- **Pagination**: List endpoints return a `{ results: [...], count: int }` envelope.
- **Error responses**: Always reference `../responses/error.yaml#/<ErrorName>` — never inline error schemas in path items.

---

## Key dependencies

| Tool                                       | Purpose                                  |
| ------------------------------------------ | ---------------------------------------- |
| `swagger-cli` (`@apidevtools/swagger-cli`) | `$ref` resolution and validation         |
| `@stoplight/spectral-cli`                  | Linting against `.spectral.yaml` ruleset |
| `yq` (mikefarah)                           | Reads `info.version` from bundle in CI   |
| `oapi-codegen`                             | Downstream Go client code generation     |
