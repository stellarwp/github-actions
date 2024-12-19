# github-actions

Collection of reusable GitHub actions and workflows.

## Workflows

### [Dependency zip](.github/workflows/dependency-zip.yml)

It allows us to download a dependency zip from S3 when available.

#### Usage

```yaml
dependency-zip:
    uses: stellarwp/github-actions/.github/workflows/dependency-zip.yml@main
    with:
      repository: <owner>/<repo>
    secrets:
      GITHUB_CHECKOUT_TOKEN: ${{ secrets.GH_BOT_TOKEN }}
      PACKAGED_ZIP_BUCKET: ${{ secrets.PACKAGED_ZIP_BUCKET }}
      S3_ACCESS_KEY_ID: ${{ secrets.S3_ACCESS_KEY_ID }}
      S3_SECRET_ACCESS_KEY: ${{ secrets.S3_SECRET_ACCESS_KEY }}
      PACKAGED_ZIP_REGION: ${{ secrets.PACKAGED_ZIP_REGION }}
      S3_ENDPOINT: ${{ secrets.S3_ENDPOINT }}
```

And then you use it in another job:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

    needs: dependency-zip

    steps:
      - name: Fetch the release zip
        if: ${{ needs.dependency-zip.outputs.use-release == 'true' }}
        uses: robinraju/release-downloader@v1.10
        with:
          repository: <owner>/<repo>
          latest: true
          fileName: <file.zip>

      - name: Download the zip from S3
        if: ${{ ! needs.dependency-zip.outputs.use-release != 'true' }}
        uses: the-events-calendar/action-s3-utility@main
        env:
          S3_BUCKET: ${{ secrets.PACKAGED_ZIP_BUCKET }}
          S3_ACCESS_KEY_ID: ${{ secrets.S3_ACCESS_KEY_ID }}
          S3_SECRET_ACCESS_KEY: ${{ secrets.S3_SECRET_ACCESS_KEY }}
          S3_REGION: ${{ secrets.PACKAGED_ZIP_REGION }}
          S3_ENDPOINT: ${{ secrets.S3_ENDPOINT }}
          COMMAND: cp
          FILE: ${{ needs.dependency-zip.outputs.s3-zip-name }}
          DESTINATION: <file.zip>
```
