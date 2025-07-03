# studio-gradle-cmake-go
The goal is to build directly from Android Studio native Golang Android application.

Trying only ARM 64bit platform. 

There is build process started in Android Studio which looks like:
[Android Studio] --> [Gradle] --> [plugin com.android.build.api.dsl.NdkBuild] --> [CMake] --> [Ninja] --> [go build]

## What works

- manual run of cmake + ninja to build .so library
```bash
cd aplikace2/app/src/main/cpp
/home/tomas/Android/Sdk/cmake/4.0.2/bin/cmake \
  -H/home/tomas/aplikace2/app/src/main/cpp \
  -DCMAKE_ANDROID_NDK=/home/tomas/Android/Sdk/ndk/26.3.11579264 \
  -DCMAKE_TOOLCHAIN_FILE=/home/tomas/Android/Sdk/ndk/26.3.11579264/build/cmake/android.toolchain.cmake \
  -DCMAKE_MAKE_PROGRAM=/home/tomas/Android/Sdk/cmake/4.0.2/bin/ninja \
  -DCMAKE_LIBRARY_OUTPUT_DIRECTORY=/home/tomas/aplikace2/app/build/intermediates/cxx/Debug/3v82m1s2/obj/arm64-v8a \
  -B/home/tomas/aplikace2/app/.cxx/Debug/3v82m1s2/arm64-v8a \
  -GNinja
```
```bash
cd /home/tomas/aplikace2/app/.cxx/Debug/3v82m1s2/arm64-v8a
ninja
```

# What do not work

JSON file linking not existing build files

In the JSON file `aplikace2/app/.cxx/Debug/3v82m1s2/arm64-v8a/.cmake/api/v1/reply/target-build-go-lib-9d2ff66b6199410e96b2.json` are links to not existing files like
`"path" : "/home/tomas/aplikace2/app/.cxx/Debug/3v82m1s2/arm64-v8a/CMakeFiles/build-go-lib",`
and file: same-path/CMakeFiles/build-go-lib.rule

Gradle ends with

`C/C++: /home/tomas/aplikace2/app/src/main/cpp/CMakeLists.txt debug|arm64-v8a : There was an error parsing CMake File API result. Please open a bug with zip of /home/tomas/aplikace2/app/.cxx/Debug/3v82m1s2/arm64-v8a`

```
* Exception is:
2025-07-01T20:51:57.638+0200 [ERROR] [org.gradle.internal.buildevents.BuildExceptionReporter] org.gradle.api.tasks.TaskExecutionException: Execution failed for task ':app:configureCMakeDebug[arm64-v8a]'.
2025-07-01T20:51:57.638+0200 [ERROR] [org.gradle.internal.buildevents.BuildExceptionReporter]   at org.gradle.api.internal.tasks.execution.ExecuteActionsTaskExecuter.lambda$executeIfValid$1(ExecuteActionsTaskExecuter.java:130)
2025-07-01T20:51:57.638+0200 [ERROR] [org.gradle.internal.buildevents.BuildExceptionReporter]   at org.gradle.internal.Try$Failure.ifSuccessfulOrElse(Try.java:293)
```
Full log in file gradle.log and https://gradle.com/s/qvqm6v4al43c2

# Tested versions:
- Gradle 8.14.2 (standalone wrapper)
- cmake 4.0.2 (from Android SDK)
- Ninja 1.12.1 (from Android SDK)
- OpenJDK 17.0.15
- Android SDK - Android 15
- Android NDK - r26d (older because of go/clang [incompatibility](https://github.com/golang/go/issues/74410) )
- Android Studio - Narwhal 2025.1.1


# Building

```bash
./gradlew clean
./gradlew assemble

./gradlew assembleDebug --debug --stacktrace --scan  # for full logs

```

# What is not working?

It looks like cmake create structure of [File API](https://cmake.org/cmake/help/latest/manual/cmake-file-api.7.html) which Gradle plugin do not understand. Probably because of missing files (CMakeFiles/build-go-lib and CMakeFiles/build-go-lib.rule) which are mentioned in target-build-go-lib-9d2ff66b6199410e96b2.json.

Cmake is confused because there are no .cpp source files. Compiling .so library with Go marked like "UTILITY" instead "SHARED_LIBRARY".

# Sources

gradle running cmake command like
```bash
/home/tomas/Android/Sdk/cmake/4.0.2/bin/cmake \
  -H/home/tomas/aplikace2/app/src/main/cpp \
  -B/home/tomas/aplikace2/app/.cxx/Debug/3v82m1s2/arm64-v8a \
  -DCMAKE_SYSTEM_NAME=Android \
  -DCMAKE_EXPORT_COMPILE_COMMANDS=ON \
  -DCMAKE_SYSTEM_VERSION=24 \
  -DANDROID_PLATFORM=android-24 \
  -DANDROID_ABI=arm64-v8a \
  -DCMAKE_ANDROID_ARCH_ABI=arm64-v8a \
  -DCMAKE_ANDROID_NDK=/home/tomas/Android/Sdk/ndk/26.3.11579264 \
  -DCMAKE_TOOLCHAIN_FILE=/home/tomas/Android/Sdk/ndk/26.3.11579264/build/cmake/android.toolchain.cmake \
  -DCMAKE_MAKE_PROGRAM=/home/tomas/Android/Sdk/cmake/4.0.2/bin/ninja \
  -DCMAKE_LIBRARY_OUTPUT_DIRECTORY=/home/tomas/aplikace2/app/build/intermediates/cxx/Debug/3v82m1s2/obj/arm64-v8a \
  -DCMAKE_RUNTIME_OUTPUT_DIRECTORY=/home/tomas/aplikace2/app/build/intermediates/cxx/Debug/3v82m1s2/obj/arm64-v8a \
  -G Ninja
```

