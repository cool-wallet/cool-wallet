# Building BeautyWallet for Android

## Requirements and Setup

The following are the system requirements to build BeautyWallet for your Android device.

```
Ubuntu >= 16.04 
Android SDK 28
Android NDK 23.2.8568313 (the beauty-wallet codebase maybe can be tweaked to use ndk 17c)
Flutter 2 or above
yacc (available with 'sudo apt install -y bison')
```

## Building BeautyWallet for Android

These steps will help you configure and execute a build of BeautyWallet from its source code.

### 1. Installing Package Dependencies

BeautyWallet cannot be built without the following packages installed on your build system.

- curl unzip automake build-essential file pkg-config git python libtool libtinfo5 cmake openjdk-8-jre-headless clang bison libncurses5

You may easily install them on your build system with the following command:

```bash
sudo apt-get install -y curl unzip automake build-essential file pkg-config git python libtool libtinfo5 cmake openjdk-8-jre-headless clang bison libncurses5
```

### 2. Installing Android Studio and Android toolchain

You may download and install the latest version of Android Studio [here](https://developer.android.com/studio#downloads).
After installing, start Android Studio, and go through the "Setup Wizard." This installs the latest Android SDK,
Android SDK Command-line Tools, and Android SDK Build-Tools, which are required by BeautyWallet.
**Be sure you are installing SDK version 28 or later when stepping through the wizard**

### 3. Installing Flutter

Need to install flutter with version `3.x.x`. For this please check section
[Install Flutter manually](https://docs.flutter.dev/get-started/install/linux#install-flutter-manually).

### 4. Verify Installations

Verify that the Android toolchain, Flutter, and Android Studio have been correctly installed on your system with the following command:

```bash
flutter doctor
```

The output of this command will appear like this, indicating successful installations.
If there are problems with your installation, they **must** be corrected before proceeding.

```
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.x.x, on Linux, locale en_US.UTF-8)
[✓] Android toolchain - develop for Android devices (Android SDK version 28)
[✓] Android Studio (version 4.0)
```

### 5. Generate a secure keystore for Android

```bash
keytool -genkey -v -keystore $HOME/key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias key
```

You will be prompted to create two passwords. First you will be prompted for the "store password",
followed by a "key password" towards the end of the creation process.
**TAKE NOTE OF THESE PASSWORDS!** You will need them in later steps. 

### 6. Acquiring the BeautyWallet Source Code

Create the directory that will be used to store the BeautyWallet source & build folders...

```bash
mkdir -pv ~/vcs/beauty-wallet-umbrella
cd ~/vcs/beauty-wallet-umbrella
```

..and download the source code into that directory.

```bash
git clone --recursive https://github.com/beauty-wallet/beauty-wallet.git --branch main
```

Proceed into the source code before proceeding with the next steps:

```bash
cd beauty-wallet/scripts/android/
```

### 7. Installing Android NDK

```bash
./install_ndk.sh
```

### 8. Execute Build & Setup Commands for BeautyWallet

We need to generate project settings like app name, app icon, package name, etc. For this need to setup
environment variables and configure project files. 

Please pick what app you want to build: cakewallet or monero.com.

```bash
source ./app_env.sh `echo -n "<cakewallet OR monero.com>"`
```
(it should be `source ./app_env.sh cakewallet` or `source ./app_env.sh monero.com`)

Then run configuration script for setup app name, app icon etc:

```bash
./app_config.sh
```  

Build the latest cmake

```bash
pushd ~/vcs
git clone --recursive https://gitlab.kitware.com/cmake/cmake.git
cd cmake
./bootstrap && make && sudo make install
popd 
```

Build the libraries and their dependencies:

```bash
./build_all.sh
```

Now the dependencies need to be copied into the BeautyWallet project with this command:

```bash
./copy_monero_deps.sh
```

It is now time to change back to the base directory of the BeautyWallet source code:

```bash
cd ../..
```

Install Flutter package dependencies with this command:

```bash
flutter pub get
```

TO DO FIX THIS: (((
    The plugin `cw_monero` uses a deprecated version of the Android embedding.
    To avoid unexpected runtime failures, or future build failures, try to see if this plugin supports the Android V2 embedding. Otherwise, consider removing it since a future release of Flutter will remove these deprecated APIs.
    If you are plugin author, take a look at the docs for migrating the plugin to the V2 embedding: https://flutter.dev/go/android-plugin-migration.
)))

Your Beauty Wallet binary will be built with cryptographic salts, which are used for secure encryption
of your data. You may generate these secret salts with the following command:

```bash
flutter packages pub run tool/generate_new_secrets.dart
```

Next, we must generate key properties based on the secure keystore you generated for Android (in step 5).
**MODIFY THE FOLLOWING COMMAND** with the "store password" and "key password" you assigned when
creating your keystore (in step 5).

```bash
flutter packages pub run tool/generate_android_key_properties.dart keyAlias=key storeFile=$HOME/key.jks storePassword=`echo -n storepassword` keyPassword=`echo -n keypassword`
```
**REMINDER:** The *above* command will **not** succeed unless you replaced the `storePassword` and `keyPassword` variables
with the correct passwords for your keystore.

Then we need to generate localization files.

```bash
flutter packages pub run tool/generate_localization.dart
```

Lastly, we will generate mobx models for the project.

Generate mobx models for `cw_core`:

```bash
cd cw_core && flutter pub get && flutter packages pub run build_runner build --delete-conflicting-outputs && cd ..
```

Generate mobx models for `cw_monero`:

```bash
cd cw_monero && flutter pub get && flutter packages pub run build_runner build --delete-conflicting-outputs && cd ..
```

Generate mobx models for `cw_bitcoin`:

```bash
cd cw_bitcoin && flutter pub get && flutter packages pub run build_runner build --delete-conflicting-outputs && cd ..
```

Generate mobx models for `cw_haven`:

```bash
cd cw_haven && flutter pub get && flutter packages pub run build_runner build --delete-conflicting-outputs && cd ..
```

Finally build mobx models for the app:

```bash
flutter packages pub run build_runner build --delete-conflicting-outputs
```

### 9. Build!

```bash
flutter build apk --release
```

Copyright (c) 2022 Cake Technologies LLC.
Copyright (c) 2023 Beauty Wallet Team. All Rights Reserved.
