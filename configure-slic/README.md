# Configure slic

The composite action prepares the environment for running the tests with [slic](https://github.com/stellarwp/slic).

## Inputs

| Name            | Required | Default | Description                                                                                                                                                                                                                                                                                   |
|-----------------|----------|---------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `wp_version`    | no       | ''      | If it's not provided the [default slic WordPress version](https://github.com/stellarwp/slic/blob/7e79022ce53adfcad514f09528fcb2d204b9e77b/.github/workflows/publish-wordpress-docker-image.yml#L19) will be used. Any value supported by `wp core update` `--version` argument can be passed. |
| `airplane_mode` | no       | 'on'    | If airplane mode enabled no external files are loaded or HTTP requests performed. To disable airplane mode pass `off`.                                                                                                                                                                        |


## Workflow example

```yaml
name: Automated Testing

on:
  pull_request:
    branches:
      - main

jobs:
  tests:
    name: Tests
    runs-on: ubuntu-latest
    steps:
      - name: Clone repository
        uses: actions/checkout@v4

      - name: Configure slic
        uses: stellarwp/github-actions/configure-slic@main
        with:
          wp_version: 6.7

      - name: Use package
        run: |
          ${SLIC_BIN} use ${{ github.event.repository.name }}

      - name: Build
        run: |
          ${SLIC_BIN} composer install
          ${SLIC_BIN} npm ci
          ${SLIC_BIN} npm run build

      - name: Run tests
        run: ${SLIC_BIN} run --ext DotReporter

      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        if: ${{ failure() }}
        with:
          name: Tests failure output
          path: ./tests/_output/**
          if-no-files-found: ignore
```
