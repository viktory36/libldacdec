### LDAC decoder

this is an early stage, jet functional LDAC audio stream decoder.

Shout-out to [@Thealexbarney](https://github.com/Thealexbarney) for the heavy lifting.
LDAC is basically a stripped down, streaming only ATRAC9.

#### Build
```sh
sudo apt install libsndfile1 libsndfile1-dev libsamplerate0 libsamplerate0-dev
make libldacdec.so
sudo make install

```

#### Usage
see ldacdec.c for example usage

#### ldacdec
takes an LDAC stream and decodes it to WAV

#### ldacenc
uses Android LDAC encoder library to create LDAC streams from audio

#### Build BlueALSA with support for this library:
```sh
LDAC_INCLUDE_DIR=/usr/include/ldac
LDAC_LIB_DIR=/usr/lib

export LDAC_ABR_CFLAGS="-I$LDAC_INCLUDE_DIR"
export LDAC_ABR_LIBS="-L$LDAC_LIB_DIR -lldacBT_abr"
export LDAC_DEC_CFLAGS="-I$LDAC_INCLUDE_DIR"
export LDAC_DEC_LIBS="-L$LDAC_LIB_DIR -lldacdec"
export LDAC_ENC_CFLAGS="-I$LDAC_INCLUDE_DIR"
export LDAC_ENC_LIBS="-L$LDAC_LIB_DIR -lldacBT_enc"


autoreconf --install
mkdir build && cd build
../configure --enable-aac --enable-ofono --enable-aptx --enable-aptx-hd --with-libopenaptx --enable-ldac --enable-debug
make
sudo make install
```

#### Building for Raspberry Pi

Apply the following patch to your kernel
```diff
diff --git a/net/bluetooth/l2cap_sock.c b/net/bluetooth/l2cap_sock.c
index eebe25610..64db1db3f 100644
--- a/net/bluetooth/l2cap_sock.c
+++ b/net/bluetooth/l2cap_sock.c
@@ -1825,7 +1825,7 @@ static void l2cap_sock_init(struct sock *sk, struct sock *parent)
                        break;
                }
 
-               chan->imtu = L2CAP_DEFAULT_MTU;
+               chan->imtu = 0;
                chan->omtu = 0;
                if (!disable_ertm && sk->sk_type == SOCK_STREAM) {
                        chan->mode = L2CAP_MODE_ERTM;
```

If you're running a Debian-based system, you can also run [this script](https://github.com/anonymix007/libldacdec/blob/master/patch-kernel.sh)
to patch and build the kernel, with this command:
```sh
# If you have cloned the repository
./patch-kernel.sh --install

# If you havent't cloned the repository
curl https://raw.githubusercontent.com/anonymix007/libldacdec/master/patch-kernel.sh | bash -s -- --install
```
If you want to add additional configurations to the kernel, you can pass `--configure`.
If you don't want to install the kernel, but you just want to compile it, you can remove `--install` from the command.

Run the following script to automatically build and install bluez-alsa with LDAC decoding capabilities:
```sh
#!/bin/sh

sudo apt install -y git automake build-essential libtool pkg-config python3-docutils
sudo apt install -y libasound2-dev libbluetooth-dev libdbus-1-dev libglib2.0-dev libsbc-dev
sudo apt install -y check libfdk-aac-dev lcov libldacbt-enc-dev libldacbt-abr-dev libbsd-dev libopenaptx-dev libunwind-dev libncurses-dev libreadline-dev libspandsp-dev libsndfile1 libsndfile1-dev libsamplerate0 libsamplerate0-dev

git clone https://github.com/anonymix007/libldacdec.git
cd libldacdec
make libldacdec.so
sudo make install
cd

git clone https://github.com/Arkq/bluez-alsa.git
cd bluez-alsa
LDAC_INCLUDE_DIR=/usr/include/ldac
LDAC_LIB_DIR=/usr/lib
export LDAC_ABR_CFLAGS="-I$LDAC_INCLUDE_DIR"
export LDAC_ABR_LIBS="-L$LDAC_LIB_DIR -lldacBT_abr"
export LDAC_DEC_CFLAGS="-I$LDAC_INCLUDE_DIR"
export LDAC_DEC_LIBS="-L$LDAC_LIB_DIR -lldacdec"
export LDAC_ENC_CFLAGS="-I$LDAC_INCLUDE_DIR"
export LDAC_ENC_LIBS="-L$LDAC_LIB_DIR -lldacBT_enc"
autoreconf --install --force
mkdir build && cd build
../configure --enable-aac --enable-aptx --enable-aptx-hd --with-libopenaptx --enable-ldac --enable-debug --enable-cli
make -j4
sudo make install
```
