on:
  workflow_dispatch:

name: Test and Coverage
jobs:
  test:
    runs-on: ubuntu-latest
    name: test and coverage
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          persist-credentials: false # otherwise, the token used is the GITHUB_TOKEN, instead of your personal access token.
          fetch-depth: 0 # otherwise, there would be errors pushing refs to the destination repository.

      - name: Setup go
        uses: actions/setup-go@v2
        with:
          go-version: "1.21.0"

      - name: Run Test
        run: |
          go test ./... -covermode=count -coverprofile=coverage.out
          go tool cover -func=coverage.out -o=coverage.out

      - name: Go Coverage Badge # Pass the `coverage.out` output to this action
        uses: tj-actions/coverage-badge-go@v2
        with:
          filename: coverage.out

      - name: Verify Changed files
        uses: tj-actions/verify-changed-files@v17
        id: verify-changed-files
        with:
          files: README.md

      - name: Commit changes
        if: steps.verify-changed-files.outputs.files_changed == 'true'
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git add README.md
          git commit -m "chore: Updated coverage badge."

      - name: Push changes
        if: steps.verify-changed-files.outputs.files_changed == 'true'
        uses: ad-m/github-push-action@master
        with:
          github_token: ${{ github.token }}
          branch: ${{ github.head_ref }}
