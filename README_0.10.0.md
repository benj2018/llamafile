This branch is a work in progress, currently aimed at replicating a cosmopolitan
llama.cpp build from scratch.

For this reason, regular build will likely fail for a while. The most up-to-date
instructions on how to replicate our builds are here.

# Building llamafile

The code is currently a hyper-simplification of mmojo-server's, with the mininum
set of changes required to rebuild llama.cpp with cosmopolitan. It is based on
llama.cpp commit a44d77126c911d105f7f800c17da21b2a5b112d1.

At the moment, the build relies on the latest compiled version of cosmocc as described
[here](https://github.com/BradHutchings/Mmojo-Server/blob/main/instructions/303-Build-Cosmopolitan.md) 
Make sure it is available and customize the `COSMO_DIR` environment variable below.

Run `make setup` to pull from the git submodules and apply the required patches.

From the llamafile repository root, run the following:

```
# set paths
export COSMO_DIR="$PWD/cosmocc" # change this to your cosmo dir
export PATH="$COSMO_DIR/bin:$SAVE_PATH"
export BUILD_PATH="$PWD/build-llamafile"

# set build commands
export AR="cosmoar"
export CC="cosmocc -I$COSMO_DIR/include -L$COSMO_DIR/lib \
    -DCOSMOCC=1 -nostdinc -O3 $EXTRA_FLAGS "
export CXX="cosmoc++ -I$COSMO_DIR/include \
    -DCOSMOCC=1 -nostdinc -nostdinc++ -O3 -Wno-format-truncation $EXTRA_FLAGS \
    -I$COSMO_DIR/include/third_party/libcxx \
    -L$COSMO_DIR/lib"

# run a fresh build
rm -rf $BUILD_PATH
cd llama.cpp
cmake -B $BUILD_PATH -DBUILD_SHARED_LIBS=OFF -DLLAMA_CURL=OFF -DLLAMA_OPENSSL=OFF \
	-DCMAKE_SYSTEM_NAME=Generic -UCMAKE_SYSTEM_PROCESSOR \
	-DCMAKE_USER_MAKE_RULES_OVERRIDE="$PWD/../cosmocc-override.cmake" \
	-DCMAKE_AR="$(command -v cosmoar)" \
	-DCMAKE_RANLIB="$(command -v cosmoranlib)"

cmake --build $BUILD_PATH --config Release -j 8
```
