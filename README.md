# setup-client

GitHub Action step for adding GPUs to your run via [LUPINE](https://github.com/lupinemachines/lupine).

It installs the prebuilt LUPINE client shims. When `LUPINE_SERVER` is unset,
it also installs the Lupine Cloud CLI and exchanges the job's GitHub Actions
OIDC identity for a short-lived Lupine token.

This lets you run workloads that need a GPU directly on GitHub Actions or any other CPU-only
actions runner.

By default, this action uses https://lupine.sh/ hosted GPUs. Set
`LUPINE_SERVER` to use a server you manage instead; that is the only connection
configuration understood by the action.

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

Before running the workflow, register `owner/repository` and its workload in the
[Lupine console](https://console.lupine.sh/github-actions) or with the CLI:

```shell
lupine oidc add owner/repository --ref main
```

Branch, pull-request target, and protected-environment policies are separate.
For example, `--pull-request main` explicitly lets PRs targeting `main`
authenticate. PR code can then consume Lupine access, including approved fork
workflows, so enable that policy only when intended.

The first run that verifies a new registration must be initiated by the same
GitHub user who registered it.

`id-token: write` lets the job request its own signed identity token; it does
not grant write access to the repository. The GitHub token is sent directly to
the Lupine coordinator for verification and is never stored.

Later steps receive `LUPINE_STATE_DIR`, so the CLI uses the short-lived
credential automatically. The Lupine API endpoint is the CLI's built-in
production default.

The OIDC identity belongs to the repository and workflow invoking this
composite action, not to `lupinemachines/setup-client`. Each caller therefore
registers its own workload policy.

By default, the action uses the latest public LUPINE release and installs the
CUDA 13.1.0 / Ubuntu 24.04 / x86_64 client asset.

This action installs the LUPINE `libcuda` and `libnvidia-ml` shim libraries. It
does not install CUDA runtime libraries, CUDA development tools, or
`nvidia-smi`; install those separately if your job needs them, we recommend [this action](https://github.com/Jimver/cuda-toolkit).

To use a server you manage, set `LUPINE_SERVER` at the job or workflow level:

```yaml
jobs:
  test:
    runs-on: ubuntu-24.04
    env:
      LUPINE_SERVER: gpu.example.com:14833
    steps:
      - uses: lupinemachines/setup-client@v1
```

In this mode the action installs and configures only the client shims. It does
not install the Lupine Cloud CLI or request a GitHub OIDC token.

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `version` | `latest` | LUPINE release tag to download, or `latest`. |
| `cuda-version` | `13.1.0` | CUDA version in the client asset name. |
| `ubuntu-version` | `24.04` | Ubuntu version in the client asset name. `ubuntu24.04` is also accepted. |

## Outputs

| Output | Description |
| --- | --- |
| `lib-dir` | Directory containing the shim libraries. |
| `libcuda` | Path to `libcuda.so.1`. |
| `libnvidia-ml` | Path to `libnvidia-ml.so.1`. |
