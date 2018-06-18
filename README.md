
# Generating-Signed-APK on Windows and Generate the Development and Release Key Hashes for Facebook.

Android requires that all apps be digitally signed with a certificate before they can be installed, so to distribute the Android application via [Google Play store](https://play.google.com/store), a signed release APK is needed.

## Requirements
* [Java](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html)
* [Android Studio](https://developer.android.com/studio/)

## Setting up Java Path

  Assuming Java is installed in `C:\Program Files\Java\jdkx.x.x_x` directory −
* Right-click on `My Computer` and select `Properties`.
* Click the `Environment variables` button under the `Advanced` tab.
* Now, click on the `Path` variable so that it also contains the path to the Java executable.
* Now add the path `C:\Program Files\Java\jdkx.x.x_x\bin`

## Generating a Signing Key

To generate a private signing key using `keytool`, on Windows `keytool` must run from

* `C:\Program Files\Java\jdkx.x.x_x\bin`. 


```
  $ keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
```

  This command prompts for passwords for the keystore and key, and to provide the Distinguished Name fields for the key. It then generates the keystore as a file called `my-release-key.keystore`.
  
  The keystore contains a single key, valid for 10000 days. The alias is a name that will use later when signing the app, one should remember to take note of the alias.
  
### Setting up Gradle Variables

1. Place the `my-release-key.keystore` file under the `android/app` directory in the project folder.
2. Edit the file `~/.gradle/gradle.properties` or `android/gradle.properties` and add the following (replace `*****` with the correct keystore password, alias and key password),

```
MYAPP_RELEASE_STORE_FILE=my-release-key.keystore
MYAPP_RELEASE_KEY_ALIAS=my-key-alias
MYAPP_RELEASE_STORE_PASSWORD=*****
MYAPP_RELEASE_KEY_PASSWORD=*****
```

These are going to be global gradle variables, which can later use in the gradle config to sign the app.

> **Note about saving the keystore:**

> Once the app is published on the Play Store, one will need to republish the app under a different package name (losing all downloads and ratings) if one wants to change the signing key at any point. So keystore must be backed up and passwords must not be forgotten.


### Adding signing config to the app's gradle config

Edit the file `android/app/build.gradle` in the project folder and add the signing config,

```
gradle
...
android {
    ...
    defaultConfig { ... }
    signingConfigs {
        release {
            if (project.hasProperty('MYAPP_RELEASE_STORE_FILE')) {
                storeFile file(MYAPP_RELEASE_STORE_FILE)
                storePassword MYAPP_RELEASE_STORE_PASSWORD
                keyAlias MYAPP_RELEASE_KEY_ALIAS
                keyPassword MYAPP_RELEASE_KEY_PASSWORD
            }
        }
    }
    buildTypes {
        release {
            ...
            signingConfig signingConfigs.release
        }
    }
}
...
```

## Generating the Development and Release Key Hashes for A App (Facebook)

To ensure the authenticity of the interactions between the app and Facebook, one need to supply us with the Android key hash for the development environment. If the app has already been published, one should add the release key hash too.

### Generating a Development Key Hash

There will be a unique development key hash for each Android Development Environment.

> In Windows, Key and Certificate Management Tool (keytool) from the Java Development Kit is needed which we have discussed above, `openssl-for-windows` openssl library for Windows from the [Google Code Archive](https://code.google.com/archive/p/openssl-for-windows/downloads).

To generate a development key hash, run the following command in a command prompt in the Java SDK folder: 

```
keytool -exportcert -alias androiddebugkey -keystore "C:\Users\USERNAME\.android\debug.keystore" | "PATH_TO_OPENSSL_LIBRARY\bin\openssl" sha1 -binary | "PATH_TO_OPENSSL_LIBRARY\bin\openssl" base64   
```

### Generating the release APK

Simply run the following in a terminal:

```
sh
$ cd android
$ ./gradlew assembleRelease
```

Gradle's `assembleRelease` will bundle all the JavaScript needed to run the app into the APK. If change is needed the way the JavaScript bundle and/or drawable resources are bundled (e.g. if the default file/folder names or the general structure of the project is changed), `android/app/build.gradle` must be seen through to see how one can update it to reflect these changes.

> Note: Make sure gradle.properties does not include _org.gradle.configureondemand=true_ as that will make release build skip bundling JS and assets into the APK.

The generated APK can be found under `android/app/build/outputs/apk/app-release.apk`, and is ready to be distributed.


