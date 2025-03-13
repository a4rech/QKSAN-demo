# QKSAN DEMO

The demo and evaluation dataset for QKSAN.

## Reproduce

Prerequirement: git and docker

Use the following commands to fetch a demo environment:

```bash
git clone https://github.com/a4rech/QKSAN-demo.git
cd QKSAN-demo
docker compose run --build --rm linux-asan-demo /bin/bash
```

### Run QNX kernel within QKSAN

Run the following command in the docker container:

```bash
python3 run.py run-qemu \
  --qemu ./qemu-kreit/build/qemu-system-x86_64 \
  --kernel ./assets/system.ifs \
  --disk ./assets/ifs.img \
  --use-qksan \
  --kernel-info ./assets/qnx-info.json \
  --accel tcg \
  --target qnx
```

You will get an interactive shell of QNX, then navigate to the path `/mnt/asan-testcases`, run the programs under this directory, you will get the QKSAN complains as below:

```text
KASAN TEST [DEBUG] Ring0 prepared, g_state: 1
QKASAN: heap-use-after-free in 0xffff800000078f75
        cpu 0 pid: 90123 (./double-free)
        try to read on address 0xffff800004e90720, size 4
Freed chunk info:
        chunk at 0xffff800004e90720, size: 20
        allocated by thread 90123 (./double-free) at 0xffff800000078fb3
        stack state when chunk allocated:
0000: ffff800000078fb8 ffff800004e2e138 0000000008048d8c ffff808000007228
0020: ffff808000007228 ffff800004e37ed8 0000000008048c33 ffff800004e1b020
0040: ffff800004e8fd10 ffff808000007000 0000000004e8fd10 ffff800004e1b020
0060: ffff8000000968f9 ffff800004e34000 ffff8000067ee020 ffff8000067ee020
0080: ffff800004e1b020 000000000000000c ffff800004e30818 0000000000000000
00a0: 0000000000001000 ffff80000008612f 0000000000000000 0000000008047d58
00c0: ffff8000067ee020 ffff8000000acc2a ffff800004e1c0c8 00000000000a26f3
00e0: 180000000000005b 0000000000000000 0000000000000000 0000000000000000
0100: ffff800004e15020 ffff800004e1b020 0000000000000000 0000000000000000
0120: ffff80000007333d 000001100010b8c0 ffff80000010b8c0 00000001002b5d98

        free by thread 0 () at 000000000000000000

...
...
```

### Detect the Bugs Found by Syzkaller

Run the following command in the docker container:

```bash
python3 ./run.py qksan \
  --qemu ./qemu-kreit/build/qemu-system-x86_64 \
  --kernel ./assets/bzImage-nokasan \
  --disk ./assets/disk.qcow2 \
  --dataset ./dataset.csv \
  --kernel-info ./assets/linux-info.json \
  --output /workspace/output/qksan-result.json \
  --worker 6
```

The program will automatically test all PoCs in the dataset and save the results in the `output` folder. (Some PoCs may not trigger sanitizers in a single run.)

### Other Testcases

See `/workspace/test.sh` and `python3 /workspace/run.py -h` for details.

