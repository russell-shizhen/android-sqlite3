# android-sqlite3
An Android project building SQLite3 c source code into static and shared libraries for NDK usage.

This project will build sqlite3 for all Android ABIs **except** mips and mips64

### Build Environment ###
* Android Studio: 2.3.3
* NDK: android-ndk-r15c
* Gradle wrapper: 3.3

### Setup ### 
Home dir of Android NDK and SDK dir need to be set as below in the *local.properties* file
~~~~
sdk.dir=/Users/<your-user-name>/Library/Android/sdk
ndk.dir=/Users/<your-user-name>/Library/Android/android-ndk-r15c
~~~~

### Command to clean
open terminal and cd to project root dir
~~~~
./gradlew clean
~~~~

### Command to build ###

open terminal and cd to project root dir
~~~~
./gradlew dist
~~~~

output will be under android-sqlite3/sqlite3-libs.

### Usage ###
In your *CMakeLists.txt* file, you can either link *sqlite3.a* or *sqlite3-shared.so* per your technical requirement. 
Note that the debug symbols of .so files have been stripped. 
