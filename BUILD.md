# How the packs are built

On Linux with the Android NDK 27.2.12479018, for API 28 (Android 9) on
arm64-v8a, and the SDK's build-tools 36.0.0 with the android-36 platform:

    BIN=$NDK/toolchains/llvm/prebuilt/linux-x86_64/bin
    CC=$BIN/aarch64-linux-android28-clang
    CXX=$BIN/aarch64-linux-android28-clang++

## Compiling

Each engine is one compiler call over its sources, run in the folder named
below, inside the engine's source archive. The engines' own makefiles are not
used, because they
reach for `-march=native`, x86 flags or the host's linker, none of which mean
anything to an ARM cross compiler.

**Weiss**, in `src/`:

    $CC -std=gnu11 -O3 -flto -DNDEBUG *.c pyrrhic/tbprobe.c tuner/*.c query/*.c \
        noobprobe/*.c onlinesyzygy/*.c -lm -o weiss

**Ethereal**, in `src/`, with its classical evaluation:

    $CC -std=gnu11 -O3 -flto -DNDEBUG -DUSE_NNUE=0 -DUSE_POPCNT *.c pyrrhic/tbprobe.c \
        -lm -o ethereal

**Vice**, in `Vice11/src/`. `stdlib.h` comes first because its `tinycthread.h`
defines a macro that clashes with C11's `once_flag` otherwise:

    $CC -include stdlib.h -O2 *.c -o vice

**Laser**, in `src/`:

    $CXX -std=c++11 -O3 -DNDEBUG -w *.cpp syzygy/*.cpp -static-libstdc++ -o laser

**Stash**, in `src/`:

    $CC -std=gnu11 -O3 -flto -DNDEBUG -Iinclude sources/*.c -lm -o stash

**Berserk**, in `src/`. The network is embedded at compile time:

    $CC -std=gnu11 -O3 -flto -DNDEBUG -DVERSION='"14"' \
        -DEVALFILE='"berserk-9b84c340af7e.nn"' \
        attacks.c bench.c berserk.c bits.c board.c eval.c history.c move.c movegen.c \
        movepick.c perft.c random.c search.c see.c tb.c thread.c transposition.c uci.c \
        util.c zobrist.c nn/accumulator.c nn/evaluate.c pyrrhic/tbprobe.c \
        -lm -o berserk

**Caissa**, in `src/`. Exceptions stay on because `Memory.hpp` throws:

    $CXX -std=c++20 -O3 -flto -funroll-loops -fno-rtti -fno-math-errno \
        -march=armv8-a+simd -DUSE_ARM_NEON -DCONFIGURATION_FINAL -DNDEBUG \
        -DCAISSA_VERSION="\"$(cat ../version.txt)\"" \
        -DCAISSA_EVALFILE='"../data/neuralNets/eval-ml-8-152B-spsa.pnn"' \
        backend/*.cpp frontend/*.cpp backend/syzygy/tbprobe.cpp -static-libstdc++ -o caissa

**RubiChess**, in `src/`. The network goes in as an object, the way its
makefile does it, with the NDK's `ld.lld` in place of the host's `ld`.
zlib's files need `unistd.h`, which glibc happens to include for them and
Android's C library does not:

    cp nn-da9c99e92a-20260819.nnue net.nnue
    $BIN/ld.lld -r -b binary -m aarch64linux net.nnue -o net.o
    for f in zlib/*.c; do $CC -O3 -w -include unistd.h -c "$f" -o "$(basename "$f" .c).o"; done
    $CXX -std=c++17 -O3 -flto -DNDEBUG -DIS_64BIT -DUSE_NEON -DUSE_ARM64 -DUSE_DOTPROD \
        -march=armv8.2-a+dotprod -DNNUEINCLUDED=net.nnue *.cpp *.o -static-libstdc++ -o rubichess

## Packing

Each engine becomes a package `org.qchess.engine.<id>` with no code. The
installer extracts its library to disk only when `extractNativeLibs` is set,
and the meta-data is how QChess knows the engine's name and file:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="org.qchess.engine.<id>"
    android:versionCode="<build date, yyyymmdd>" android:versionName="<version>">
    <uses-sdk android:minSdkVersion="28" android:targetSdkVersion="35"/>
    <application android:label="QChess engine: <name>" android:hasCode="false"
        android:extractNativeLibs="true" android:allowBackup="false">
        <meta-data android:name="org.qchess.engine.name" android:value="<name>"/>
        <meta-data android:name="org.qchess.engine.version" android:value="<version>"/>
        <meta-data android:name="org.qchess.engine.library" android:value="lib<id>.so"/>
        <meta-data android:name="org.qchess.engine.elo" android:value="<strength>"/>
        <meta-data android:name="org.qchess.engine.source" android:value="<repository>@<commit>"/>
    </application>
</manifest>
```

Then:

    mkdir -p pack/lib/arm64-v8a pack/assets
    $BIN/llvm-strip -o pack/lib/arm64-v8a/lib<id>.so <id>
    cp <the engine's licence file> pack/assets/LICENSE
    printf '<name> <version>\nBuilt from <repository> at commit <commit>.\n' > pack/assets/SOURCE
    aapt2 link -o base.apk --manifest AndroidManifest.xml \
        -I $ANDROID_SDK/platforms/android-36/android.jar -A pack/assets
    # add pack/lib/arm64-v8a/lib<id>.so to base.apk as lib/arm64-v8a/lib<id>.so (deflated)
    zipalign -f -p 4 base.apk aligned.apk
    apksigner sign --ks <keystore> --out QChess-engine-<id>.apk aligned.apk

The licence files are `COPYING.txt` (Weiss), `COPYING` (Laser), `copying`
(RubiChess) and `LICENSE` (the rest).

Android installs an update over a pack only when it is signed with the same
key. A pack built and signed elsewhere installs once the one on the phone has
been removed.

## Left out

Rodent builds for ARM but plays nonsense there (a1b1 from the starting
position). Halogen rewrites its network for the target's SIMD with a tool the
build compiles for that target, so a cross build needs the tool built twice.
Stormphrax needs ARMv8.4 and has not been tried. Seer and Igel are x86 only.
