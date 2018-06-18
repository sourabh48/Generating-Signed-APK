
# Generating-Signed-APK

Android requires that all apps be digitally signed with a certificate before they can be installed, so to distribute your Android application via [Google Play store](https://play.google.com/store), you'll need to generate a signed release APK.

## Installation

### Requirements
* [Java](http://www.oracle.com/technetwork/java/javase/downloads/jre8-downloads-2133155.html)
* [Android Studio](https://developer.android.com/studio/)

### Setting up Java Path

  Assuming Java is installed in `C:\Program Files\Java\jdkx.x.x_x` directory −
* Right-click on `My Computer` and select `Properties`.
* Click the `Environment variables` button under the `Advanced` tab.
* Now, click on the `Path` variable so that it also contains the path to the Java executable.
* Now add the path `C:\Program Files\Java\jdkx.x.x_x\bin`

### Generating a Signing Key

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


