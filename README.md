# setup-client

GitHub Action step for adding GPUs to your run via [LUPINE](https://github.com/lupinemachines/lupine).

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
    steps:
      - uses: actions/checkout@v6

      - uses: lupinemachines/setup-client@v1
        with:
          cuda-version: 13.1.0
          ubuntu-version: "24.04"

      - run: |
          echo "$LUPINE_LIBCUDA"
          ls -l "$LUPINE_LIB_DIR"
```

By default, the action uses the latest public LUPINE release and installs the
CUDA 13.1.0 / Ubuntu 24.04 / x86_64 client asset.

This action installs the LUPINE `libcuda` and `libnvidia-ml` shim libraries. It
does not install CUDA runtime libraries, CUDA development tools, or
`nvidia-smi`; install those separately if your job needs them, we recommend [this action](https://github.com/Jimver/cuda-toolkit).

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

## Outputs

| Output | Description |
| --- | --- |
| `install-dir` | Directory containing the installed release files. |
| `lib-dir` | Directory containing the shim libraries. |
| `libcuda` | Path to `libcuda.so.1`. |
| `libnvidia-ml` | Path to `libnvidia-ml.so.1`. |
| `asset-name` | Release asset downloaded by the action. |
| `download-url` | URL used to download the release asset. |
