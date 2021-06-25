## Compiling and installing

### On Linux and Mac

First install the required tools

On Ubuntu and Debian:

```
sudo apt-get install git gcc make automake libtool libreadline-dev -y
```

On CentOS:

```
sudo yum install git gcc make automake libtool readline-devel
```

On Mac:

```
brew install git gcc make automake libtool readline
```

Then copy and paste this on a terminal:

```
# Install libuv

git clone --depth=1 https://github.com/libuv/libuv
cd libuv
./autogen.sh
./configure
make
sudo make install
sudo ldconfig
cd ..

# Install binn

git clone --depth=1 https://github.com/liteserver/binn
cd binn
make
sudo make install
cd ..

# Install libsecp256k1-vrf

git clone --depth=1 https://github.com/aergoio/secp256k1-vrf
cd secp256k1-vrf
./autogen.sh
./configure --disable-benchmark
make
sudo make install
cd ..

# Install OctoDB

git clone --depth=1 https://gitlab.com/octodb/octodb
cd octodb
make
sudo make install
cd -
```


### On Windows using MinGW

Copy and paste this on a MSYS2 MinGW terminal:

```
# Compile libuv

git clone --depth=1 https://github.com/libuv/libuv
cd libuv
./autogen.sh
./configure
make
make install
cd ..

# Compile binn

git clone --depth=1 https://github.com/liteserver/binn
cd binn
make
make install
cd ..

# Compile libsecp256k1-vrf

git clone --depth=1 https://github.com/aergoio/secp256k1-vrf
cd secp256k1-vrf
./autogen.sh
./configure --with-bignum=no --disable-benchmark
make
make install
cd ..

# Compile OctoDB

git clone --depth=1 https://gitlab.com/octodb/octodb
cd octodb
make
make install
```


### For Android

Generate the native libraries with this command:

```
make android
```


### For iOS

Generate the native libraries with this command:

```
make ios
```



Automated Tests
---------------

These tests simulate many nodes on your computer

Before running the tests you will need to increase the limit of open files on your terminal:

```
ulimit -Sn 16000
```

Then you can run the automated tests with:

```
make test
```

For printing debug messages to a log file you must recompile the library in debug mode before running the tests:

```
make clean
make debug
```
