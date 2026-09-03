# Xiaomi Pad 5 (nabu) four-speaker audio

This branch is based on `sm8150-mainline/linux` commit
`5181e1358ddd6ea8028e841d928942373e6aebc8` (`v6.14.11-sm8150`).

It adds the kernel-side pieces used to bring up all four CS35L41 speaker
amplifiers on Xiaomi Pad 5 with per-amplifier CSPL protection tuning:

- describe the four amplifier tuning files in the nabu device tree;
- preload and route all four amplifiers through their DSP paths;
- load a bounded comma-separated signed 32-bit tuning parameter list;
- write the parameter buffer to ADSP2 Y memory and CSPL controls to X memory;
- reapply tuning after every DSP mailbox resume, including a preloaded DSP;
- configure the nabu speaker playback link for four channels.

The vendor WMFW, per-speaker coefficient/calibration files, tuning parameter
files, and userspace UCM configuration are not included in this repository.
They must be supplied separately for the exact device. Do not enable high
speaker gain without validated calibration and protection data.

## Build

Use an AArch64 cross compiler and an out-of-tree build directory. The tested
kernel release was built with `CONFIG_LOCALVERSION=-nabu-tmm-dsp-prot-test5`:

```sh
make O=../build ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- olddefconfig
make O=../build ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- -j4 \
  vmlinux modules Image.gz qcom/sm8150-xiaomi-nabu.dtb
```

The complete test5 tree compiled successfully and booted on nabu. The earlier
test4 revision passed a zero-sample DSP firmware, calibration, tuning, and
heartbeat check on all four amplifiers. Runtime validation of the test5 resume
retuning change should be completed before treating this branch as production
ready.
