# Build srsRAN 4G UE / RAN with ZeroMQ by disabling RF plugins
[srsRAN_4G](https://github.com/srsran/srsRAN_4G) software suite includes a virtual radio which uses the ZeroMQ networking library to transfer radio samples between applications.
Therefore, in order to confirm the facilities of EPC, I will describe the simple procedure for building these virtual eNodeB and UE instead of the real devices.
**Note that the RF plugin is disabled and the ZeroMQ library is built into the virtual eNodeB and UE.**

Please refer to the following for building srsRAN 4G UE / RAN with ZeroMQ.
- srsRAN 4G - https://docs.srsran.com/projects/4g/en/latest/

The specification of the VM that have been confirmed to work is as follows.
| OS | CPU (Min) | Memory (Min) | HDD (Min) | GCC version |
| --- | --- | --- | --- | --- |
| Ubuntu 22.04 | 1 | 2GB | 10GB | 11 (default) |
| Ubuntu 24.04 | 1 | 2GB | 10GB | 11 (downgrade from 13) |

**2GB or more memory is required to build.**

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

---

<a id="toc"></a>

## Table of Contents

- [Install the required libraries including ZeroMQ](#install_libs)
  - [Prepare for building on Ubuntu 24.04](#build_on_ubuntu_2404)
- [Clone srsRAN_4G](#clone_srsran)
- [Build srsRAN 4G UE / RAN by disabling RF plugins](#build)
- [Create configuration files of eNodeB](#create_enb_config)
- [Create the configuration file of UE](#create_ue_config)
- [Create the configuration file of NR-UE](#create_nr_ue_config)
  - [Add a Slice configuration](#add_slice)
- [Note for ensuring that packets pass through UE-RAN-UPF path](#packets_path)
- [Issues](#issues)
- [Confirmed Version List](#ver_list)
- [Sample Configurations](#sample_conf)
  - [For 5G](#5g_conf)
  - [For 4G](#4g_conf)
- [Changelog (summary)](#changelog)

---

<a id="install_libs"></a>

## Install the required libraries including ZeroMQ

```
# apt install build-essential cmake libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev libzmq3-dev
```

<a id="build_on_ubuntu_2404"></a>

### Prepare for building on Ubuntu 24.04

If building srsRAN_4G on Ubuntu 24.04, downgrade the gcc version from the default 13 to 11.

First, install gcc-11 and g++-11 in addition to the packages installed above.
```
# apt install gcc-11 g++-11
```
Then, use `update-alternatives` tool to be able to switch between gcc-11 and 13.
```
# update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 11 --slave /usr/bin/g++ g++ /usr/bin/g++-11
# update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-13 13 --slave /usr/bin/g++ g++ /usr/bin/g++-13
```
Finally, switch to using gcc-11.
```
# update-alternatives --config gcc
There are 2 choices for the alternative gcc (providing /usr/bin/gcc).

  Selection    Path             Priority   Status
------------------------------------------------------------
* 0            /usr/bin/gcc-13   13        auto mode
  1            /usr/bin/gcc-11   11        manual mode
  2            /usr/bin/gcc-13   13        manual mode

Press <enter> to keep the current choice[*], or type selection number: 1
update-alternatives: using /usr/bin/gcc-11 to provide /usr/bin/gcc (gcc) in manual mode
```
Confirm that gcc version has been switched to 11.
```
# update-alternatives --config gcc
There are 2 choices for the alternative gcc (providing /usr/bin/gcc).

  Selection    Path             Priority   Status
------------------------------------------------------------
  0            /usr/bin/gcc-13   13        auto mode
* 1            /usr/bin/gcc-11   11        manual mode
  2            /usr/bin/gcc-13   13        manual mode

Press <enter> to keep the current choice[*], or type selection number: ^C
```
```
# gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/11/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none:amdgcn-amdhsa
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 11.4.0-9ubuntu1' --with-bugurl=file:///usr/share/doc/gcc-11/README.Bugs --enable-languages=c,ada,c++,go,brig,d,fortran,objc,obj-c++,m2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-11 --program-prefix=x86_64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-bootstrap --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-vtable-verify --enable-plugin --enable-default-pie --with-system-zlib --enable-libphobos-checking=release --with-target-system-zlib=auto --enable-objc-gc=auto --enable-multiarch --disable-werror --enable-cet --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --enable-multilib --with-tune=generic --enable-offload-targets=nvptx-none=/build/gcc-11-ZcnBzW/gcc-11-11.4.0/debian/tmp-nvptx/usr,amdgcn-amdhsa=/build/gcc-11-ZcnBzW/gcc-11-11.4.0/debian/tmp-gcn/usr --without-cuda-driver --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu --with-build-config=bootstrap-lto-lean --enable-link-serialization=2
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 11.4.0 (Ubuntu 11.4.0-9ubuntu1) 
```

<a id="clone_srsran"></a>

## Clone srsRAN_4G

```
# git clone https://github.com/srsran/srsRAN_4G.git
```

<a id="build"></a>

## Build srsRAN 4G UE / RAN by disabling RF plugins

Configure that **RF plugins** is disabled to directly link the ZeroMQ library into the virtual eNodeB and UE.
```
# cd srsRAN_4G
# mkdir build
# cd build
# cmake ../ -DENABLE_RF_PLUGINS=OFF
# make -j`nproc`
```

<a id="create_enb_config"></a>

## Create configuration files of eNodeB

```
# cd srsRAN_4G/srsenb
# cp enb.conf.example ../build/srsenb/enb.conf
# cp rr.conf.example ../build/srsenb/rr.conf
# cp rb.conf.example ../build/srsenb/rb.conf
# cp sib.conf.example ../build/srsenb/sib.conf
```
Then, edit according to your environment.

<a id="create_ue_config"></a>

## Create the configuration file of UE

```
# cd srsRAN_4G/srsue
# cp ue.conf.example ../build/srsue/ue.conf
```
Then, edit according to your environment.

<a id="create_nr_ue_config"></a>

## Create the configuration file of NR-UE

When used as 5G NR-UE with ZeroMQ support, it can connect to srsRAN_Project 5G RAN with ZeroMQ.
For 5G NR-UE configuration, get `UE config` of [ZeroMQ-based Setup](https://docs.srsran.com/projects/project/en/latest/tutorials/source/srsUE/source/index.html#zeromq-based-setup) as the original file.
Also, see [here](https://github.com/s5uishida/build_srsran_5g_zmq) for how to build this RF simulated gNodeB.
```
# cd srsRAN_4G/build/srsue
# wget <link of "UE config">
```
For reference, `ue_zmq.conf` on 2023.12.07 is as follows.
```
[rf]
freq_offset = 0
tx_gain = 50
rx_gain = 40
srate = 23.04e6
nof_antennas = 1

device_name = zmq
device_args = tx_port=tcp://127.0.0.1:2001,rx_port=tcp://127.0.0.1:2000,base_srate=23.04e6

[rat.eutra]
dl_earfcn = 2850
nof_carriers = 0

[rat.nr]
bands = 3
nof_carriers = 1
max_nof_prb = 106
nof_prb = 106

[pcap]
enable = none
mac_filename = /tmp/ue_mac.pcap
mac_nr_filename = /tmp/ue_mac_nr.pcap
nas_filename = /tmp/ue_nas.pcap

[log]
all_level = info
phy_lib_level = none
all_hex_limit = 32
filename = /tmp/ue.log
file_max_size = -1

[usim]
mode = soft
algo = milenage
opc  = 63BFA50EE6523365FF14C1F45F88737D
k    = 00112233445566778899aabbccddeeff
imsi = 001010123456780
imei = 353490069873319

[rrc]
release = 15
ue_category = 4

[nas]
apn = srsapn
apn_protocol = ipv4

[gw]
netns = ue1
ip_devname = tun_srsue
ip_netmask = 255.255.255.0

[gui]
enable = false

```
Then, edit according to your environment.

<a id="add_slice"></a>

### Add a Slice configuration

The following SST/SD values are in decimal notation.
For example, SST=0x1 and SD=0x010203 are expressed in decimal as follows.
```
[slicing]
enable = true
nssai-sst = 1
nssai-sd = 66051
```

<a id="packets_path"></a>

## Note for ensuring that packets pass through UE-RAN-UPF path

Make the following settings on UE to ensure that packets pass through `UE-RAN-UPF` path.
```
# ip route change default dev tun_srsue
```
Then, for example when run iPerf3 client on UE, do the following.
```
# iperf3 -B <UE IP address> -c <IP address of iperf3 server>
```
**In this case, in order to ensure that the traffic goes through `tun_srsue` interface, UE must not directly connect to the same network as the IP address to which iPerf3 server binds.  
Note. According to [this](https://github.com/srsran/srsRAN_4G/discussions/1445#discussioncomment-11986515), srsUE is not intended to be used in high-throughput scenarios. It seems that srsUE is useful for confirming the functionality of 4G/5G core networks.**

<a id="issues"></a>

## Issues

1. If you're having an issue with a registration request from NR-UE with `[slicing]` section in the configuration to Open5GS, [this](https://github.com/srsran/srsRAN_4G/pull/1214) might be helpful.

<a id="ver_list"></a>

## Confirmed Version List

I simply confirmed the operation of the following versions.

| Version | Commit | Date | Issues |
| --- | --- | --- | -- |
| 23.11+ | `ec29b0c1ff79cebcbe66caa6d6b90778261c42b8` | 2024.02.01 | 1 |
| 23.11 | `eea87b1d893ae58e0b08bc381730c502024ae71f` | 2023.11.23 | 1 |
| 23.04.1 | `fa56836b14dc6ad7ce0c3484a1944ebe2cdbe63b` | 2023.06.19 | 1 |

<a id="sample_conf"></a>

## Sample Configurations

<a id="5g_conf"></a>

### For 5G

- [Open5GS 5GC & srsRAN 5G with ZeroMQ UE / RAN Sample Configuration](https://github.com/s5uishida/open5gs_5gc_srsran_sample_config)
- [free5GC 5GC & srsRAN 5G with ZeroMQ UE / RAN Sample Configuration](https://github.com/s5uishida/free5gc_srsran_sample_config)

<a id="4g_conf"></a>

### For 4G

- [Open5GS EPC & srsRAN 4G with ZeroMQ UE / RAN Sample Configuration](https://github.com/s5uishida/open5gs_epc_srsran_sample_config)

<a id="changelog"></a>

## Changelog (summary)

- [2025.05.11] Added instructions for building on Ubuntu 24.04.
- [2024.03.25] Updated a list of confirmed versions.
- [2023.12.02] Updated a list of confirmed versions.
- [2023.10.10] Added a list of confirmed versions.
- [2023.08.10] Added 5G NR-UE configuration file.
- [2023.05.06] Initial release.
