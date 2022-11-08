# Sourcery GitHub Action

## Overview

Run [Sourcery](https://sourcery.ai/) as a GitHub Action that you can easily add to your
Continuous Integration.

This action performs the following operations:

* Installs Sourcery on your worker machine
* Logs in to Sourcery using your access token
* Reviews your code

## Usage

### Check your entire codebase

To run Sourcery over your entire codebase whenever a push is done to the `main` branch:

```yaml
name: Check codebase using Sourcery

on:
  push:
    branches: [main]

jobs:
  review-with-sourcery:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - uses: sourcery-ai/action@v1
        with:
          token: ${{ secrets.SOURCERY_TOKEN }}
```

We recommend you store your Sourcery token as a
[GitHub secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets) on
your repository. In the example above, that secret was stored as `SOURCERY_TOKEN`.

You can retrieve your Sourcery token from your
[dashboard](https://sourcery.ai/dashboard/profile).

### Check your PR

To run Sourcery only on the changed code in a PR:

```yaml
name: Check PR using Sourcery

on: pull_request

jobs:
  review-with-sourcery:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - uses: actions/setup-python@v4
        with:
          python-version: '3.10'

      - uses: sourcery-ai/action@v1
        with:
          token: ${{ secrets.SOURCERY_TOKEN }}
          diff_ref: ${{ github.event.pull_request.base.sha }}
```

Note here that we pass the `fetch-depth: 0` option to
[`actions/checkout@v3`](https://github.com/actions/checkout). This is necessary because
Sourcery needs access to both the current branch and the main branch in order to compute
diffs.

## Inputs

### `token`

> **Type**: string
>
> **CLI equivalent**: `--token <token>`

A Sourcery token. You can retrieve yours at the
[Sourcery dashboard](https://sourcery.ai/dashboard/profile).

We recommend you to store your token as a
[GitHub secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets).

### `target`

> **Type**: string
>
> **Default**: `.`
>
> **CLI equivalent**: `--target <target>`

File(s) or directory(ies) for Sourcery to review.

This defaults to `"."`, meaning that Sourcery will review your entire project.

To review only the directories `dir1/` and `dir2/` along with the files `file1.py` and
`file2.py`, pass them as space-separated strings:

```yaml
- uses: sourcery-ai/action@v1
  with:
    token: ${{ secrets.SOURCERY_TOKEN }}
    target: dir1 dir2 file1.py file2.py
```

### `version`

> **Type**: string
>
> **Default**: The latest Sourcery version available
>
> **CLI equivalent**: None

The Sourcery CLI version to use.

This defaults to the latest version available.

To choose a specific version (let's say `v0.12.12`):

```yaml
- uses: sourcery-ai/action@v1
  with:
    token: ${{ secrets.SOURCERY_TOKEN }}
    version: 0.12.12
```

We recommend you _not_ to this option unless strictly necessary. Pinning a Sourcery
version may leave you out from the awesome features we are working on - as well as
bugfixes ;)

### `verbose`

> **Type**: either `true` or `false`
>
> **Default**: `true`
>
> **CLI equivalent**: `--verbose`

Enable or disable Sourcery's verbose output.

### `check`

> **Type**: either `true` or `false`
>
> **Default**: `true`
>
> **CLI equivalent**: `--check`

Whether Sourcery should return an error code or not if issues are found in the code.

This defaults to `true`, and hence the Sourcery CI step will fail in case Sourcery finds
issues in your code. You can pass `false` to prevent that behavior.

### `in_place`

> **Type**: either `true` or `false`
>
> **Default**: `false`
>
> **CLI equivalent**: `--in-place`

Whether Sourcery should automatically fix and modify the reviewed files in-place or not.

### `config`

> **Type**: string
>
> **Default**: None
>
> **CLI equivalent**: `--config <config-path>`

Path to a valid Sourcery configuration file.

By default, this searches the current directory and its parents for a `.sourcery.yaml`
file.

To use a configuration file located elsewhere (for instance, at
**`config_files/.my-team-rules.yaml`**):

```yaml
- uses: sourcery-ai/action@v1
  with:
    token: ${{ secrets.SOURCERY_TOKEN }}
    config: config_files/.my-team-rules.yaml
```

### `diff_ref`

> **Type**: string
>
> **Default**: None
>
> **CLI equivalent**: `--diff="git diff <diff_ref>"`

A reference to compute a `git diff`.

This is not used by default. In this case, Sourcery will run over the entire
[`target`](#target) files.

To run Sourcery only on the changed lines in a PR, use:

```yaml
- uses: sourcery-ai/action@v1
  with:
    token: ${{ secrets.SOURCERY_TOKEN }}
    diff_ref: ${{ github.event.pull_request.base.sha }}
```
