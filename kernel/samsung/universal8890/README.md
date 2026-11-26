# Exynos 8890 Android 17 kernel compatibility

The series includes real CAP_PERFMON (38), CAP_BPF (39) and
CAP_CHECKPOINT_RESTORE (40) support, per-UID CPU accounting, and bounded-loop
verification fixes needed by Android's power-supply socket filter.

Original authors are preserved for all upstream backports. CPU time-in-state
and concurrency accounting come from Google's Android 4.4 implementation,
including Sultan Alsawaf's lifetime/leak fixes. The integration enables
CONFIG_CPU_FREQ_TIMES on herolte and hero2lte and adapts output to this tree's
single-character seq_file delimiter API. Framework readers can use the legacy
/proc/uid_time_in_state and /proc/uid_concurrent_{active,policy}_time fallback.
The optional BPF-based CPU tracking backend remains unavailable with this
kernel's existing tracing configuration.

The verifier evaluates constant comparisons and supported constant ALU
operations precisely. Its existing state cache assumes an acyclic control-flow
graph, so programs containing loops are verified without that optimization.
Instruction and state-stack limits are unchanged. Nonterminating or overly
complex paths fail verification; this does not claim full modern BPF support.

Device validation on herolte, kernel 4.4.302-g57af758d2bf2, SELinux enforcing:

- The installed filterPowerSupplyEvents.o loads and HealthLoop attaches it at
  boot. All 52 verifier/traffic checks pass, including signed/unsigned branches,
  ALU values, a bounded loop, rejection of infinite loops and invalid reads,
  power-supply acceptance, non-power event rejection and boundary packets.
- All three CPU proc files have the expected formats: 31 frequency columns,
  eight active counters and two four-core policies. A two-thread workload
  produces approximately 12 seconds of UID CPU time, and UID removal cleans
  up frequency and concurrency entries.
- Capability permission regression checks pass (21 pass, three configuration
  dependent skips). Chromium browser retesting is recorded in local artifacts.

The test boot image changes only the kernel; its ramdisk and device-tree
payload are byte-identical to the previously installed image. A complete
pre-flash BOOT partition backup is retained locally.

This series is already included in 8890q's lineage-24.0 kernel branch. The
generic apply.sh intentionally skips these archived kernel patches.

For a clean kernel at dcadf09ba0cd4215c5e1ad4bff2bf6edc4f85db2 only:

```sh
git -C kernel/samsung/universal8890 am \
    "$PWD"/legacy_support_patches/kernel/samsung/universal8890/0*.patch
```

patches.json records the base/target trees, commit IDs, patch IDs and SHA-256
hashes. All 19 patches replay in an isolated Git index to the exact target tree.
