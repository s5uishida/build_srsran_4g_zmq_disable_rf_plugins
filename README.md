# Build srsRAN 4G UE / RAN with ZeroMQ by disabling RF plugins
srsRAN 4G software suite includes a virtual radio which uses the ZeroMQ networking library to transfer radio samples between applications.
Therefore, in order to confirm the facilities of EPC, I will describe the simple procedure for building these virtual eNodeB and UE instead of the real devices.
**Note that the RF plugin is disabled and the ZeroMQ library is built into the virtual eNodeB and UE.**

Please refer to the following for building srsRAN 4G UE / RAN with ZeroMQ.
- srsRAN 4G - https://docs.srsran.com/projects/4g/en/latest/

**2GB or more memory is required to build.**

---

<h2 id="install_libs">Install the required libraries including ZeroMQ</h2>

```
apt install build-essential cmake libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev libzmq3-dev
```

<h2 id="clone_srsran">Clone srsRAN_4G</h2>

```
git clone https://github.com/srsran/srsRAN_4G.git
```

<h2 id="build">Build srsRAN 4G UE / RAN by disabling RF plugins</h2>

Configure that **RF plugins** is disabled to directly link the ZeroMQ library into the virtual eNodeB and UE.
```
cd srsRAN_4G
mkdir build
cd build
cmake ../ -DENABLE_RF_PLUGINS=OFF
make
```

<h2 id="create_enb_config">Create configuration files of eNodeB</h2>

```
cd srsRAN_4G/srsenb
cp enb.conf.example ../build/srsenb/enb.conf
cp rr.conf.example ../build/srsenb/rr.conf
cp rb.conf.example ../build/srsenb/rb.conf
cp sib.conf.example ../build/srsenb/sib.conf
```
Then, edit according to your environment.

<h2 id="create_ue_config">Create configuration files of UE</h2>

```
cd srsRAN_4G/srsue
cp ue.conf.example ../build/srsue/ue.conf
```
Then, edit according to your environment.

<h2 id="create_nr_ue_config">Create configuration files of NR-UE</h2>

When used as 5G NR-UE with ZeroMQ support, it can connect to srsRAN_Project 5G RAN with ZeroMQ.
For 5G NR-UE configuration, get `UE config` of [ZeroMQ-based Setup](https://docs.srsran.com/projects/project/en/latest/tutorials/source/srsUE/source/index.html#zeromq-based-setup) as the original file.
Also, see [here](https://github.com/s5uishida/build_srsran_5g_zmq) for how to build this RF simulated gNodeB.
```
cd srsRAN_4G/build/srsue
wget <link of "UE config">
```
For reference, `ue_zmq.conf` on 2023.04.24 is as follows.
```
[rf]
freq_offset = 0
tx_gain = 50
rx_gain = 40
srate = 11.52e6
nof_antennas = 1

device_name = zmq
device_args = tx_port=tcp://127.0.0.1:2001,rx_port=tcp://127.0.0.1:2000,base_srate=11.52e6

[rat.eutra]
dl_earfcn = 2850
nof_carriers = 0

[rat.nr]
bands = 3
nof_carriers = 1

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

---

<h2 id="changelog">Changelog (summary)</h2>

- [2023.08.10] Added 5G NR-UE configuration file.
- [2023.05.06] Initial release.
