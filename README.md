# Build srsRAN 4G UE / RAN with ZeroMQ by disabling RF plugins
srsRAN 4G software suite includes a virtual radio which uses the ZeroMQ networking library to transfer radio samples between applications.
Therefore, in order to confirm the facilities of EPC, I will describe the simple procedure for building these virtual eNodeB and UE instead of the real devices.
**Note that the RF plugin is disabled and the ZeroMQ library is built into the virtual eNodeB and UE.**

Please refer to the following for building srsRAN 4G UE / RAN with ZeroMQ.
- srsRAN 4G - https://docs.srsran.com/projects/4g/en/latest/

---

<h2 id="install_libs">Install the required libraries including ZeroMQ</h2>

```
apt install build-essential cmake libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev libzmq3-dev
```

<h2 id="clone_srsran">Clone srsRAN_4G</h2>

```
git clone https://github.com/srsran/srsRAN_4G.git
```

<h2 id="edit_config">Edit srsRAN_4G/CMakeLists.txt for disabling RF plugins</h2>

Configure that **RF plugins** is disabled to directly link the ZeroMQ library into the virtual eNodeB and UE.
```diff 
--- CMakeLists.txt.orig 2023-05-02 10:51:19.942559736 +0900
+++ CMakeLists.txt      2023-05-02 10:51:43.645754492 +0900
@@ -69,7 +69,7 @@
 option(AUTO_DETECT_ISA       "Autodetect supported ISA extensions"      ON)
 
 option(ENABLE_GUI            "Enable GUI (using srsGUI)"                ON)
-option(ENABLE_RF_PLUGINS     "Enable RF plugins"                        ON)
+option(ENABLE_RF_PLUGINS     "Enable RF plugins"                        OFF)
 option(ENABLE_UHD            "Enable UHD"                               ON)
 option(ENABLE_BLADERF        "Enable BladeRF"                           ON)
 option(ENABLE_SOAPYSDR       "Enable SoapySDR"                          ON)
```

<h2 id="build">Build srsRAN 4G UE / RAN</h2>

```
cd srsRAN_4G
mkdir build
cd build
cmake ../
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
