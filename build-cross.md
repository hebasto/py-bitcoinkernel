# Notes about Cross-compiling

Here is an example of cross-compiling for Windows:

```
make -C depend/bitcoin/depends NO_QT=1 NO_ZMQ=1 NO_WALLET=1 NO_USDT=1 NO_LIBEVENT=1 HOST=x86_64-w64-mingw32
cmake -B build --toolchain depend/bitcoin/depends/x86_64-w64-mingw32/toolchain.cmake
```
