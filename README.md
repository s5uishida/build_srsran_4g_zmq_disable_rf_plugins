# Build srsRAN_4G UE / RAN with ZeroMQ by disabling RF plugins
[srsRAN_4G](https://github.com/srsran/srsRAN_4G) software suite includes a virtual radio which uses the ZeroMQ networking library to transfer radio samples between applications.
Therefore, in order to confirm the facilities of EPC, I will describe the simple procedure for building these virtual eNodeB and UE instead of the real devices.
**Note that the RF plugin is disabled and the ZeroMQ library is built into the virtual eNodeB and UE.**

Please refer to the following for building srsRAN_4G UE / RAN with ZeroMQ.
- srsRAN_4G - https://docs.srsran.com/projects/4g/en/latest/

The specification of the VM that have been confirmed to work is as follows.
| OS | CPU (Min) | Memory (Min) | HDD (Min) |
| --- | --- | --- | --- |
| Ubuntu 24.04 | 1 | 2GB | 10GB |

**2GB or more memory is required to build.**  
**Note. Please see [here](https://docs.srsran.com/projects/4g/en/latest/dev_status.html) for the development status of srsRAN_4G.**

---

### [Sample Configurations and Miscellaneous for Mobile Network](https://github.com/s5uishida/sample_config_misc_for_mobile_network)

---

<a id="toc"></a>

## Table of Contents

- [Install the required libraries including ZeroMQ](#install_libs)
- [Clone srsRAN_4G](#clone_srsran)
- [Build srsRAN_4G UE / RAN by disabling RF plugins](#build)
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

<a id="clone_srsran"></a>

## Clone srsRAN_4G

```
# git clone https://github.com/srsran/srsRAN_4G.git
```

<a id="build"></a>

## Build srsRAN_4G UE / RAN by disabling RF plugins

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
| 25.10 | `6bcbd9e5bf8686aa7085202cd847c5ddd64a9c16` | 2026.01.18 | -- |
| 23.11+ | `1fab3df863f66fdb6c3b34f1b39e745dbcb12d5e` | 2025.10.22 | 1 |
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

- [Open5GS EPC & srsRAN_4G with ZeroMQ UE / RAN Sample Configuration](https://github.com/s5uishida/open5gs_epc_srsran_sample_config)

<a id="changelog"></a>

## Changelog (summary)

- [2026.01.26] Updated a list of confirmed versions.
- [2025.12.15] Added a link to the srsRAN_4G development status at the top of this article.
- [2025.12.13] Updated a list of confirmed versions.
- [2025.05.11] Added instructions for building on Ubuntu 24.04.
- [2024.03.25] Updated a list of confirmed versions.
- [2023.12.02] Updated a list of confirmed versions.
- [2023.10.10] Added a list of confirmed versions.
- [2023.08.10] Added 5G NR-UE configuration file.
- [2023.05.06] Initial release.
