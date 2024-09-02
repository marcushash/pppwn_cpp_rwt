# PPPwn c++

This is the C++ rewrite of the [PPPwn](https://github.com/TheOfficialFloW/PPPwn) exploit.

# Features

- Reduced binary size
- Support for a broad range of CPU architectures and systems
- Improved performance on Windows (more precise sleep times)
- Automatic restart upon failure
- Can be compiled as a library for integration into your application

# Nightly build

You can download the latest build from [nightly.link](https://nightly.link/xfangfang/PPPwn_cpp/workflows/ci.yaml/main?status=completed).

For Windows users:
- Install [npcap](https://npcap.com) before running this program.
- There are various GUI wrappers for `pppwn_cpp`; using them is recommended if you are not comfortable with the command line.

For macOS users:
- Run `sudo xattr -rd com.apple.quarantine <path-to-pppwn>` after downloading.
- Refer to [#10](https://github.com/xfangfang/PPPwn_cpp/issues/10) for more details.

# Usage

### show help

```shell
pppwn
```

### list interfaces

```shell
pppwn list
```

### run the exploit

```shell
pppwn --interface en0 --fw 1100 --stage1 "stage1.bin" --stage2 "stage2.bin" --timeout 10 --auto-retry
```

- `-i` `--interface`: the network interface which connected to ps4
- `--fw`: the firmware version of the target ps4 (default: `1100`)
- `-s1` `--stage1`: the path to the stage1 payload (default: `stage1/stage1.bin`)
- `-s2` `--stage2`: the path to the stage2 payload (default: `stage2/stage2.bin`)
- `-t` `--timeout`: the timeout in seconds for ps4 response, 0 means always wait (default: `0`)
- `-wap` `--wait-after-pin`: the waiting time in seconds after first round CPU pinning (default: `1`)
- `-gd` `--groom-delay`: wait for 1ms every `groom-delay` rounds during Heap grooming (default: `4`)
- `-bs` `--buffer-size`: PCAP buffer size in bytes, less than 100 indicates default value (usually 2MB) (default: `0`)
- `-a` `--auto-retry`: automatically retry when fails or timeout
- `-nw` `--no-wait-padi`: don't wait one more [PADI](https://en.wikipedia.org/wiki/Point-to-Point_Protocol_over_Ethernet#Client_to_server:_Initiation_(PADI)) before starting the exploit
- `-rs` `--real-sleep`: use CPU for more precise sleep time (Only used when execution speed is too slow)
- `--web`: use the web interface
- `--url`: the url of the web interface (default: `0.0.0.0:7796`)

Supplement:

1. For `--timeout`, the waiting period for `PADI` is excluded, allowing you to start `pppwn_cpp` before the PS4 is fully launched.
2. By default, `pppwn_cpp` waits for two `PADI` requests to improve stability (see [TheOfficialFloW/PPPwn/pull/48](https://github.com/TheOfficialFloW/PPPwn/pull/48)). You can disable this feature with the `--no-wait-padi` parameter if it's not needed.
3. Setting `--wait-after-pin` to `20` may improve stability, as suggested by [SiSTR0/PPPwn/pull/1](https://github.com/SiSTR0/PPPwn/pull/1). This option is not available in the web interface.
4. The `--groom-delay` value is empirical. While the Python version of `pppwn` does not include waits during heap grooming, the C++ version may require it to avoid kernel panics on some PS4 models. You can set any value between 1 and 4097 (where 4097 means no wait).
5. Adjust the `--buffer-size` to manage memory usage on low-end devices. For example, setting it to `10240` uses about 3MB of memory, though very small values might lead to incomplete packet capture.

# Development

This project depends on libpcap. By default, CMake will search for it in the system path. To compile libpcap from source (useful for cross-compiling), you can add the CMake option -DUSE_SYSTEM_PCAP=OFF.

For more information, please refer to the workflow file .github/workflows/ci.yaml.
```shell
# native build (macOS, Linux)
cmake -B build
cmake --build build -t pppwn

# cross compile for mipsel linux (soft float)
cmake -B build -DZIG_TARGET=mipsel-linux-musl -DUSE_SYSTEM_PCAP=OFF -DZIG_COMPILE_OPTION="-msoft-float"
cmake --build build -t pppwn

# cross compile for arm linux (armv7 cortex-a7)
cmake -B build -DZIG_TARGET=arm-linux-musleabi -DUSE_SYSTEM_PCAP=OFF -DZIG_COMPILE_OPTION="-mcpu=cortex_a7"
cmake --build build -t pppwn

# cross compile for Windows
# https://npcap.com/dist/npcap-sdk-1.13.zip
cmake -B build -DZIG_TARGET=x86_64-windows-gnu -DUSE_SYSTEM_PCAP=OFF -DPacket_ROOT=<path to npcap sdk>
cmake --build build -t pppwn
```

# Credits

Big thanks to FloW for the magical work and to xfangfang—you're my heroes.
