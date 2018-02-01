# Introduction
Since you're here, I will assume you want to get started playing around with a version 2.0 TPM.

For those unfamiliar, the TPM provides out-of-band general cryptographic, storage, policy
and key management operations (among other things).

There are generally 5 components when using a TPM.
1. The TPM itself, this can be a firmware TPM, a discreet TPM chip or a simulator
2. The optional but recommended resource manager (RM), which manages object life cycles in the TPM.
   Examples of this are the userspace RM called abrmd and the in-linux-kernel abrmd in the tpm driver
   itself. RMs implement the same interface as a TPM.
3. The low-level system api called sapi.
4. The transmission interface library called tcti
5. A client using sapi, like the tpm2-tools project.

When wishes to access the TPM, it will:
1. Create a tcti context. This is how it sends data to and from a TPM or Resource Manager(RM).
2. Provide that tcti context when initializes the sapi.
3. Use the sapi interfaces to send commands to the TPM.
   * The sapi builds command arrays and send them via the tcti to the TPM or RM.
   * The sapi then parses the response array for error codes and other response information.

The tools are clients, that perform those three steps.

# Installing

Start off by satisfying all dependencies. Depending on what
version of the tpm2-tools is being used, you may need different
versions. Please consult the [https://github.com/tpm2-software/tpm2-tools/wiki/Dependency-Matrix](dependency matrix).

please install:
  - [tss](https://github.com/tpm2-software/tpm2-tss)
  - (optional) [pandoc](https://pandoc.org/) (required for manpage generation)
  - (optional) lcov (required for configure option: --enable-code-coverage)
  - (optional) [abrmd](https://github.com/tpm2-software/tpm2-abrmd) (required for abrmd)

For example, on ubuntu, installing the master version of everything:
```
# install package manager deps for tools
sudo apt-get install lcov pandoc autoconf-archive

# install package manage deps for tss
sudo apt-get install liburiparser-dev

# install package manage deps for abrmd
sudo apt-get install libdbus-1-dev libglib2.0-dev

# install TSS itself
git clone https://github.com/tpm2-software/tpm2-tss.git
cd tpm2-tss
./bootstrap
./configure --enable-unit
make check
sudo make install

# Install abrmd itself
git clone https://github.com/tpm2-software/tpm2-abrmd.git
cd tpm2-abrmd
./bootstrap
./configure --enable-unit --with-dbuspolicydir=/etc/dbus-1/system.d
make check
sudo make install

# Install tools itself
git clone https://github.com/tpm2-software/tpm2-tools.git
cd tpm2-abrmd
./bootstrap
./configure --enable-unit
make check
sudo make install
```
Note: we added **--enable-unit** to **configure** and **check** to **make**,
so that one can observe the unit tests passing.

Now that you have the tss, abrmd and the tools installed, we can run hello world.

## TPM Dependency
If you have a hardware TPM, that's great. If you don't, don't fret, there is a simulator
available.

We don't recommend testing, development or learning against a real TPM.
For example, you could inadvertently lock something in NV ram that is very difficult/impossible to erase
or wear out NV ram with too many write cycles. We reccomend using the simulator, and
hello world will expect it and abrmd to be in place.

### Installing the TPM2.0 Simulator:
```
wget https://downloads.sourceforge.net/project/ibmswtpm2/ibmtpm974.tar.gz
mkdir ibmtpm974
cd ibmtpm974
tar -xavf ../ibmtpm974.tar.gz
cd src
make
```
### Starting the TPM2.0 Simulator:
Now, while still in the src directory from section *Installing the TPM2.0 Simulator*

```
./tpm2_server &
TPM command server listening on port 2321
Platform server listening on port 2322
```
That command will start the server listening on tcp/ip ports 2321 and 2322.

Now that we have a TPM to send commands too, we need a resource manager (RM). TPMs
have very limited resources, so RM's are like virtual memory managers, and help
multiple clients use the TPM without stepping on each others toes. There are
caveats with RMs, but we won't discuss those here, and instead focus on the basics.

With that said, lets start the RM:
```
tpm2-abrmd --allow-root
```
abrmd is designed by default to connect to the tpm2_server that was started previously.

# Hello World

With the tpm2_server and the abrmd running, we can run a tool to get the pcr values
of a tpm. Running the ```tpm2_pcrlist``` command should yield a result like below:
```
tpm2_pcrlist 
sha1 :
  0  : 0000000000000000000000000000000000000003
  1  : 0000000000000000000000000000000000000000
  2  : 0000000000000000000000000000000000000000
  3  : 0000000000000000000000000000000000000000
  4  : 0000000000000000000000000000000000000000
  5  : 0000000000000000000000000000000000000000
<snip>
```