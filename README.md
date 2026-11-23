# Cosmos emulator branching: cold CI comparison

Use **Actions → Compare three isolated emulators → Run workflow** on `main`.
Before benchmarking, CI selects the `cosmos-agent-branching` Cargo package
and runs an integration test that imports `cosmos_agent_branching`. A rename
failure stops the job before any emulator is started.
The default fixture is **100,000 orders per worker**. Each fresh emulator
loads its own copy (300,000 seed writes); native branches seed once (100,000)
and fork three siblings. Both use three concurrent workers and the same 600
SDK operations. The job runs fresh/overlay/overlay/fresh on one machine and
publishes a side-by-side table in the Actions job summary. Compare **tasks
verified** for pipeline latency, not SDK ops/s. Baseline reuse is not measured.

## Dedicated runner required

No workflow runs on push or pull request. No fallback to a GitHub-hosted runner.
Use a dedicated trusted Windows x64 runner with the custom label
`cosmos-overlay-benchmark`; **do not attach your development workstation**.
Configure the runner account with PowerShell 7, Rust 1.99+ (MSVC and linker),
Python (`azure-cosmos==4.17.1` importable under `python -I`), and Docker Desktop
running Linux containers with `desktop-linux` and WSL2. Leave `CARGO_TARGET_DIR`
unset. Allow Docker at least 10 CPUs, 30 GiB total and 24 GiB **available** before
the run. These are admission thresholds, not measured peak memory usage.
Pre-pull the pinned emulator image on that runner:

```powershell
docker --context desktop-linux pull mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator@sha256:db53613dae19d0a3d5ad50688d14aa6e957af082dafea56b52e1bd3a6b3f1406
```

Set the demo repo Actions secret `LIBRARY_READ_TOKEN` to a token with read-only
Contents access to the private `TheovanKraay/cosmos-agent-swarm` library. The
workflow checks out a pinned revision and does not persist credentials. It
needs no Azure account or cloud database key. Only dispatch once the separate
runner is online. If a job is interrupted, inspect its labelled Docker
resources on that runner before repeating; do not run a global prune.