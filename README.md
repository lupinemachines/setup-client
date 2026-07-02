# setup-action

GitHub Action for installing the prebuilt LUPINE client shims from
`lupinemachines/lupine` release assets.

The action downloads a release ZIP such as
`lupine-client-cuda-13.1.0-ubuntu24.04-x86_64.zip`, verifies `SHA256SUMS` when
present, and exports the library paths needed by later workflow steps.

## Usage

```yaml
jobs:
  test:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v6

      - uses: lupinemachines/setup-action@v1
        with:
          version: v0.2.0
          cuda-version: 13.1.0
          ubuntu-version: "24.04"
          server: demo.lupinemachines.com:14833

      - run: |
          echo "$LUPINE_LIBCUDA"
          ls -l "$LUPINE_LIB_DIR"
```

By default, the action uses the latest public LUPINE release and installs the
CUDA 13.1.0 / Ubuntu 24.04 / x86_64 client asset.

This action installs the LUPINE `libcuda` and `libnvidia-ml` shim libraries. It
does not install CUDA runtime libraries, CUDA development tools, or
`nvidia-smi`; install those separately if your job needs them.

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

## Exported Environment

When `export-env` is `true`, later workflow steps receive:

- `LUPINE_HOME`
- `LUPINE_LIB_DIR`
- `LUPINE_LIBCUDA`
- `LUPINE_LIB`
- `LD_LIBRARY_PATH`
- `LUPINE_SERVER`, when the `server` input is set

## Supported Assets

The action currently supports Linux `x86_64` assets published by
`lupinemachines/lupine`, for example:

- `lupine-client-cuda-13.1.0-ubuntu24.04-x86_64.zip`
- `lupine-client-cuda-12.4.1-ubuntu22.04-x86_64.zip`
