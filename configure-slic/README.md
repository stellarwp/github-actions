# Configure slic

The composite action prepares the environment for running the tests with [slic](https://github.com/stellarwp/slic).

## Inputs

| Name               | Required | Default | Description                                                                                                                                                                                                                                                                                   |
|--------------------|----------|---------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `slic_ref`         | no       | 'main'  | Git reference to checkout slic repository. It might be needed for debug or testing purposes.                                                                                                                                                                                                  |
| `php_version`      | no       | ''      | If it's not provided the [default slic PHP version](https://github.com/stellarwp/slic/blob/f89dbeaa7af9b795ede5d43be0fea3f8d929fd4a/.env.slic#L50) will be used. Only single dot notation is supported (e.g. 8.1, not 8.1.31).                                                                |
| `composer_version` | no       | ''      | If it's not provided the [default slic Composer version](https://github.com/stellarwp/slic/blob/1bdf39f39a57f2228a80a7870880fbdaec53a66d/src/commands/composer.php#L48) will be used. 1 or 2 options can be passed.                                                                           |
| `wp_version`       | no       | ''      | If it's not provided the [default slic WordPress version](https://github.com/stellarwp/slic/blob/7e79022ce53adfcad514f09528fcb2d204b9e77b/.github/workflows/publish-wordpress-docker-image.yml#L19) will be used. Any value supported by `wp core update` `--version` argument can be passed. |
| `airplane_mode`    | no       | 'on'    | If airplane mode enabled no external files are loaded or HTTP requests performed. To disable airplane mode pass `off`.                                                                                                                                                                        |


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

      - name: Configure php
        uses: shivammathur/setup-php@v2
        with:
          php-version: '7.4'
          coverage: none
          tools: stellarwp/pup

      - name: Configure Node.js
        uses: actions/setup-node@v4
        with:
          node-version-file: '.nvmrc'
          cache: 'npm'

      - name: Build
        run: |
          pup build --dev

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
