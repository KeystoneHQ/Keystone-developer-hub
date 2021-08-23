
# Keystone Application Upgrade Package Verification

## Introduction
Keystone offers a method for verifying official release upgrade packages. You can compare a version package you compiled from Github source code with the official release update package to accomplish this.

Refer to the below sections for instructions.

## Download official release update package
- [Firmware upgrades on the official Keystone website](https://keyst.one/firmware)

## Verify Checksum

Calculate the sha256 checksum of the package to ensure that it is consistent with the official website 
```
shasum -a 256 V1.2.0.zip
```

## Unzip official release update package
  1. Unzip the file you just downloaded and get a file called update.zip
  
  2. Use the public key defined in the code to unzip upgrade package update.zip
  
  [public key for multi-coin version](https://github.com/KeystoneHQ/keystone-cold-app/blob/master/app/build.gradle#L112) 
  
  [public key for btc-only version](https://github.com/KeystoneHQ/keystone-cold-app-btc/blob/master/app/build.gradle#L112) 
  
  An upgrade package consists of the following parts:
- `app_[version_code]_[version_name]_[git_commit_id]_[apk_sha1_checksum].apk` : Keystone cold upgrade version package
- `manifest.json` : Upgrade package digest information
- `serial_*.bin` : Keystone Secure Element upgrade version package
- `signed.rsa` : Signature for upgrade package
  
3. Verify the Signature

`signed.rsa` is the digital signature of `manifest.json` by SHA1withRSA, you can verify the signature with public key      mentioned above. Refer to the code [verify update.zip signature](https://github.com/KeystoneHQ/Keystone-cold-app/blob/8ddba8ba7f9937132dd73ff489e3297cc01ec7ae/app/src/main/java/com/keystone/cold/update/Checking.java#L114)
  

## Download source code
- Download the code of the branch corresponding to the official upgrade package version from Github. For example, if the official upgrade package version is `V1.1.0`, the corresponding code branch is `V1.1.0-release`.
Run the following command to download the code:

`git clone -b [code branch] git@github.com:KeystoneHQ/keystone-cold-app.git --recursive`
or
`git clone -b [code branch] git@github.com:KeystoneHQ/keystone-cold-app-btc.git --recursive`

Replace `[code branch]` with a specific branch name.

- When the code is downloaded you can run `git log` to find the latest commit ID.
  This commit ID should be the same as the commit ID obtained in the previous step.

## Build APK

`./gradlew assembleVault_v2Release`

You can find the APK in:

 `./releases/[version_code]/app_***.apk`

## Verify APK
We now have two APK files, one extracted from official upgrade package, another built from the open source code.

Because the keystore used to sign the APK cannot be made public, a test keystore is used in the open source code.
Therefore, the digest of the APK file cannot be directly compared. Instead, we can compare whether the information before signing is consistent.

Change the suffix of the APK file to .zip, and then unzip the .zip file.
You can find the `META-INF/CERT.SF` file.

Then compare the two `META-INF/CERT.SF` files. If they are the same，you will have verified that the official upgrade package matches the open source code.







