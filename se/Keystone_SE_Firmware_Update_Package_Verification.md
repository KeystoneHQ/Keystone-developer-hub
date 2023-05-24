# Keystone Secure Element Firmware Upgrade Package Verification

## Introduction
Keystone offers a method for verifying official release upgrade packages. You can compare a version package you compiled from Github source code with the official release update package to accomplish this.

We have built the shell script for the verification. you can check the file [here](https://github.com/KeystoneHQ/keystone-se-firmware/blob/bitcoin/verify.sh). Before run the sctipt, make sure the docker is installed.


Refer to the below sections for instructions.

## Download official release update package
- [Firmware upgrades on the official Keystone website](https://keyst.one/firmware)

## Unzip official release update package
Use the public key defined in the code to unzip upgrade package update.zip
  
  [public key for all-coin version](https://github.com/KeystoneHQ/keystone-cold-app/blob/master/app/build.gradle#L112) 
  
  [public key for btc-only version](https://github.com/KeystoneHQ/keystone-cold-app-btc/blob/master/app/build.gradle#L112) 
  
An upgrade package consists of the following parts:
- `app_[version_code]_[version_name]_[git_commit_id]_[apk_sha1_checksum].apk` : Keystone cold upgrade version package
- `manifest.json` : Upgrade package digest information
- `serial_[version]_*.bin` : Keystone Secure Element upgrade version package
- `signed.rsa` : Signature for upgrade package

## Download source code
Download the code which is the corresponding to the official upgrade package version from Github. For example, if the official upgrade package version is `v1.5.0`, you can get the tag v1.5.0 for the commit.
Run the following command to download the code:

```
  git clone git@github.com:KeystoneHQ/keystone-se-firmware.git
  git checkout v1.5.0
```

## Build secure element firmware
Build with ARM IDEs like "Keil MDK V4.x".   
1. Download Keil MDK4.x [here](https://www.keil.com/demo/eval/armv4.htm).
2. Install MDK and register license.
3. Run MDK, and add firmware project. Open the dialog `Project - Open Project`, select `mason.uvproj` in directory `keystone-se-firmware/`.
4. Build the firmware project. Select the dialog `Project - Rebuild all target files` to compile the source files.
5. Find the firmware image `mason_app.hex` and `mason_app.bin` in directory `keystone-se-firmware/`.

## Make upgrade package
  Run the shell script `verify.sh` from the project root folder, after execution, you can find the `unsigned_firmware_*.bin` file which is the unsigned package.
 
The unsigned package is consist of the following parts:
| Index | Part | Length | Description
|:-------:|:-----------------:|:-----------------:|:-----------------
1 | Header | 128Bytes   | version info and signature for verifying
2 | Body   | 528Bytes*n | version package encrypted and checksum

Update Package Header's length is 128 bytes. It is consist of the following parts:
| Index | Item | Length | Description
|:-------:|:-----------------:|:-----------------:|:-----------------
1 | ver            | 4Bytes     | version BCD encode
2 | ver_checksum   | 4Bytes     | front 4 bytes of ver sha256
3 | reserve        | 24Bytes    | reserve
4 | body_hash      | 32Bytes    | sha256 value of entire body
5 | signature      | 64Bytes    | signature of body_hash

Update Package Body has several 528 bytes blocks. Each block is consist of the following parts:
| Index | Item |  Sub Item| Length | Description
|:-------:|:-----------------:|:-----------------:|:-----------------:|:-----------------
1 | block_content  |  block_addr   | 8Bytes   | flash address of block (3des encrypt)
2 | block_content  |  block_bin    | 512Bytes | package bin of block (3des encrypt)
2 | bloc_checksum  |  /            | 8Bytes   | front 8 bytes of block_content(3des encrypt) sha256

## Verify update package
After compare the update package with the `serial_*.bin` unzipped from official release update package,
You will find the "signature" in update Package Header is different.

Beyond that, the other parts should be same.
It could prove the official release update package was built by Github source code.