# llama.cpp Android Tutorial

llama.cpp link： https://github.com/ggerganov/llama.cpp

## Termux installation

Official Website: [termux](https://termux.dev/en/index.html).

Change repo for faster speed (optional):

```bash
termux-change-repo
```

Check [here](https://wiki.termux.com/wiki/Package_Management) for more help.

## Install necessary code and packages

Download following packages in termux:

```bash
pkg i clang wget git cmake
```

Obtain llama.cpp source code:

```bash
git clone https://github.com/ggerganov/llama.cpp.git
```


## Importing language model

Type`termux-setup-storage` in termux terminal before importing model. Grant access for termux so that user could access files outside of termux. For details, please visit: https://wiki.termux.com/wiki/Termux-setup-storage

Use `adb push` command to import:

```
adb push \path\to\your\model\on\windows /storage/emulated/0/download
```

`~/storage/downloads` in termux home directory shares download files on Android system. Move it to `~/llama.cpp/models` 

```bash
mv ~/storage/downloads/model_name ~/llama.cpp/models
```

## Compile and build

**Strongly recommend to use cmake rather than make**

### Based on Android NDK（Non-OpenCL）

#### Install Pre-build NDK

Location: [https://github.com/lzhiyong/termux-ndk](https://github.com/lzhiyong/termux-ndk/releases)
Copy the download link from there and download the zip with wget:

```bash
wget https://github.com/lzhiyong/termux-ndk/releases/download/[NDK_ZIP_FILE].zip
```

Unzip and set NDK PATH:

```bash
unzip [NDK_ZIP_FILE].zip
export NDK=~/[EXTRACTED_NDK_PATH]
```

#### Build

Build under `~/llama.cpp/build` (Change the flags according to your Android version and CPU):

```bash
mkdir build
cd build
cmake -DCMAKE_TOOLCHAIN_FILE=$NDK/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-24 -DCMAKE_C_FLAGS=-march=native ..
make
```

Run:

```bash
cd bin/
./main YOUR_PARAMETERS
```

### Based on OpenCL + CLBlast（Recommend）

Download necessary packages: 

```bash
apt install ocl-icd opencl-headers opencl-clhpp clinfo libopenblas
```

Manually compile CLBlast and copy `clblast.h` into llama.cpp:

```bash
git clone https://github.com/CNugteren/CLBlast.git
cd CLBlast
cmake .
cmake --build . --config Release
mkdir install
cmake --install . --prefix ~/CLBlast/install
cp libclblast.so* $PREFIX/lib
cp ./include/clblast.h ../llama.cpp
```

Copy OpenBLAS files to llama.cpp:

```bash
cp /data/data/com.termux/files/usr/include/openblas/cblas.h .
cp /data/data/com.termux/files/usr/include/openblas/openblas_config.h .
```

#### Build llama.cpp with CLBlast

```bash
cd ~/llama.cpp
mkdir build
cd build
cmake -DLLAMA_CLBLAST=ON -DCMAKE_TOOLCHAIN_FILE=$NDK/build/cmake/android.toolchain.cmake -DANDROID_ABI=arm64-v8a -DANDROID_PLATFORM=android-24 -DCMAKE_C_FLAGS=-march=native -DCLBlast_DIR=~/CLBlast/install/lib/cmake/CLBlast ..
cd ..
make
```

Add `LD_LIBRARY_PATH` under `~/.bashrc`（Run program directly on physical GPU）：

```bash
echo "export LD_LIBRARY_PATH=/vendor/lib64:$LD_LIBRARY_PATH:$PREFIX" >> ~/.bashrc
```

Check GPU is available for OpenCL:

```bash
clinfo -l
```

If everything works fine, for Qualcomm Snapdragon SoC, it will display:

```bash
Platform #0: QUALCOMM Snapdragon(TM)
 `-- Device #0: QUALCOMM Adreno(TM)
```

Run:

```bash
cd bin/
./server -m models/[YOUR_MODEL].gff
```
Open [http://127.0.0.1:8080/] http://127.0.0.1:8080/ in a web browser of your choice.

#### Results

- SoC: Qualcomm Snapdragon 8 Gen 2
- RAM: 16 GB
- Model: llama-2-7B-Chat-Q4_0.gguf（[Download](https://huggingface.co/Rabinovich/Llama-2-7B-Chat-GGUF)）
- Multiple long conversations
- Params:
  - Context size = 4096
  - Batch size = 16
  - Threads = 4

Result:

- Load time = 1129.17 ms
- 3.67 tokens per second
