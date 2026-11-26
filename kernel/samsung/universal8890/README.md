# Exynos 8890 modern capabilities

These five patches preserve the original upstream authors and backport
CAP_PERFMON (38), CAP_BPF (39) and CAP_CHECKPOINT_RESTORE (40), their permission
checks and SELinux mappings.

Android 17's native zygote enumerates capabilities through 40. The original
kernel exposes 37; querying capability 38 returns EINVAL and aborts Chromium's
child before browser initialization. This series adds real kernel permissions.
It retains the SYS_ADMIN fallback needed by Android bpfloader, separates BPF
from pointer-disclosing monitoring privileges and adds no unsupported BPF
instruction or verifier features.

The series is already included in the 8890q kernel's lineage-24.0 branch. It is
archived here for review/replay and is intentionally not applied by the generic
apply.sh: applying it again to that branch would duplicate kernel commits.

For a clean kernel at dcadf09ba0cd4215c5e1ad4bff2bf6edc4f85db2 only:

```sh
git -C kernel/samsung/universal8890 am \
    "$PWD"/legacy_support_patches/kernel/samsung/universal8890/000*.patch
```

patches.json records base/target trees, commit IDs, patch IDs and SHA-256 hashes.
All five patches were replayed against the base in an isolated Git index and
produced the exact target tree. Kernel build/boot and Chromium retesting on the
new kernel are still required.
