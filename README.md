# Cosmos emulator branching: cold CI comparison

## One-command rename check (GitHub-hosted)

From any directory with the GitHub CLI signed in, run:

```sh
gh workflow run rename-smoke.yml -R TheovanKraay/cosmos-agent-branching-demo
```

This tests the pinned Cargo package and its Rust import on GitHub's Ubuntu
runner. It uses the existing `LIBRARY_READ_TOKEN`; no Docker, emulator,
self-hosted runner, or local build is needed. See the
[smoke test runs](https://github.com/TheovanKraay/cosmos-agent-branching-demo/actions/workflows/rename-smoke.yml)
for the result.

## Separate emulator benchmark

Use **Actions → Compare three isolated emulators → Run workflow** on `main`.
Run the separate rename smoke test above first if you want to verify
private checkout and compilation without starting any emulators.
Start with **10,000 orders per worker** (the default), then select 100,000 if
both approaches pass. Each fresh emulator loads its own copy (three times
the selected record count); native branches seed once
and fork three siblings. Both use three concurrent workers and the same 600
SDK operations. The job runs fresh/overlay/overlay/fresh on one machine and
publishes a side-by-side table in the Actions job summary. Compare **tasks
verified** for pipeline latency, not SDK ops/s. Baseline reuse is not measured.

```sh
gh workflow run compare.yml -R TheovanKraay/cosmos-agent-branching-demo -f records=10000
# After the 10k job passes:
gh workflow run compare.yml -R TheovanKraay/cosmos-agent-branching-demo -f records=100000
```

Both workflows are manual, run on GitHub-hosted `ubuntu-24.04`, and require
no local emulator or runner registration. The benchmark pulls the pinned
emulator image and runs all four arms sequentially on the same VM. Minimum
available RAM and free disk are sampled once per second during each arm;
these are host-level readings, not per-emulator allocation measurements.
An unsupported OverlayFS mount or resource failure fails the job rather
than changing the test or falling back to a full copy.

The demo can be public while the library remains private. Keep
`LIBRARY_READ_TOKEN` in Actions secrets, limited to read-only Contents on
`TheovanKraay/cosmos-agent-swarm`. Both workflows check out a pinned
revision without persisting credentials; neither publishes library source,
raw emulator logs, or build artifacts. Only trusted maintainers should be
able to change the workflows. No Azure account is needed.