# setup-client

GitHub Action step for adding GPUs to your run via [LUPINE](https://github.com/lupinemachines/lupine).

It installs the prebuilt LUPINE client shims and Lupine Cloud CLI. By default,
it also authenticates the CLI without a stored API key by exchanging the job's
GitHub Actions OIDC identity for a short-lived Lupine token.

This lets you run workloads that need a GPU directly on GitHub Actions or any other CPU-only
actions runner.

By default, this action uses https://lupine.sh/ hosted free GPUs, however you can point
it at your self-hosted GPU with the `server` argument.

Only Linux runners are supported.

## Usage

```yaml
jobs:
  test:
    runs-on: ubuntu-24.04
    permissions:
      contents: read
      id-token: write
    steps:
      - uses: actions/checkout@v6

      - uses: lupinemachines/setup-client@v1
        with:
          cuda-version: 13.1.0
          ubuntu-version: "24.04"

      - run: |
          lupine status
          echo "$LUPINE_LIBCUDA"
          ls -l "$LUPINE_LIB_DIR"
```

Before running the workflow, register `owner/repository` and its trusted branch
in the [Lupine console](https://console.lupine.sh/github-actions). The first run
that verifies a new registration must be initiated by the same GitHub user who
registered it.

`id-token: write` lets the job request its own signed identity token; it does
not grant write access to the repository. The GitHub token is sent directly to
the Lupine coordinator for verification and is never stored.

Later steps receive `LUPINE_API_URL` and `LUPINE_STATE_DIR`, so they use the
same endpoint and short-lived credential automatically.

By default, the action uses the latest public LUPINE release and installs the
CUDA 13.1.0 / Ubuntu 24.04 / x86_64 client asset.

This action installs the LUPINE `libcuda` and `libnvidia-ml` shim libraries. It
does not install CUDA runtime libraries, CUDA development tools, or
`nvidia-smi`; install those separately if your job needs them, we recommend [this action](https://github.com/Jimver/cuda-toolkit).

To install the CLI without authenticating, set `authenticate: "false"`. This is
useful for public commands such as `lupine gpus`; authenticated cloud commands
will still require `lupine login`.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `version` | `latest` | LUPINE release tag to download, or `latest`. |
| `repository` | `lupinemachines/lupine` | Repository that publishes the release assets. |
| `cuda-version` | `13.1.0` | CUDA version in the client asset name. |
| `ubuntu-version` | `24.04` | Ubuntu version in the client asset name. `ubuntu24.04` is also accepted. |
| `architecture` | `auto` | Release asset architecture. Currently resolves to `x86_64`. |
| `install-dir` | runner tool cache | Directory where the client files should be installed. |
| `github-token` | empty | Optional token for private repositories or higher rate limits. |
| `server` | empty | Optional `LUPINE_SERVER` value exported for later steps. |
| `export-env` | `true` | Export `LUPINE_*` and `LD_LIBRARY_PATH` variables. |
| `cli-version` | `latest` | Lupine Cloud CLI version to install. |
| `cli-install-dir` | runner tool cache | Directory where the CLI is installed. |
| `api-url` | `https://api.lupine.sh` | Coordinator URL used as the OIDC audience and API endpoint. |
| `authenticate` | `true` | Authenticate the CLI with the current job's GitHub Actions identity. |

## Outputs

| Output | Description |
| --- | --- |
| `install-dir` | Directory containing the installed release files. |
| `lib-dir` | Directory containing the shim libraries. |
| `libcuda` | Path to `libcuda.so.1`. |
| `libnvidia-ml` | Path to `libnvidia-ml.so.1`. |
| `asset-name` | Release asset downloaded by the action. |
| `download-url` | URL used to download the release asset. |
| `cli-path` | Path to the installed Lupine Cloud CLI. |
