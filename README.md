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

---

<h2 id="changelog">Changelog (summary)</h2>

- [2023.05.06] Initial release.
