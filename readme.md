# FCM Expo notifications with bare workflow

This guide helps you start a [expo bare workflow](https://docs.expo.dev/introduction/managed-vs-bare/#bare-workflow) project and setup [expo push notifications](https://docs.expo.dev/push-notifications/overview/).

First [initialize an aplication with bare worklow](https://docs.expo.dev/bare/hello-world/)

    expo init expo-notifications-bare-workflow

Test the application

    cd expo-notifications-bare-workflow
    expo run:android

When the application is working we can approach to setup Firebase. Inside Google Console, setup your firebase project https://console.firebase.google.com/u/1/.
After the project is successfully created, register your application to the new project under **Cloud Messaging** inside the console. You will need your package name which can be obtained from android/app/src/main/AndroidManifest.xml, look for package="your-package-name". Download the provided **google-services.js**.

### Let's configure the project

Modify the project-level build.gradle file (./android/build.gradle)

    buildscript {
        repositories {
            // Check that you have the following line (if not, add it):
            google()  // Google's Maven repository
        }

        dependencies {
            ...
            // Add this line
            classpath 'com.google.gms:google-services:4.3.10'
        }
    }

    allprojects {
    ...
        repositories {
            // Check that you have the following line (if not, add it):
            google()  // Google's Maven repository
            ...
        }
    }

Modify the app-level build.gradle file (./android/app/build.gradle)

    apply plugin: 'com.android.application'
    // Add this line
    apply plugin: 'com.google.gms.google-services'
    dependencies {
      // Import the Firebase BoM
      implementation platform('com.google.firebase:firebase-bom:28.4.2')
      // Add the dependencies for the desired Firebase products
      // https://firebase.google.com/docs/android/setup#available-libraries
    }

> The actual package name or version may differ in time, so change then accordingly as firebase walkthrough suggest

**Copy** the google-services.js to android/app folder!

Register a handler to receive the FCM push token:

    import { StyleSheet, Text, View } from 'react-native';

    import * as Notifications from "expo-notifications";

    Notifications.getDevicePushTokenAsync()
        .then((pushToken) => {
        console.log("pushtoken", pushToken);
        })
        .catch((err) => console.warn("getDevicePushTokenAsync", err));


        export default function App() {

Install expo-notifications package.

    expo install expo-notifications

Before we continue, kill the app a start it again.
You should see error message like:

> Encountered an exception while calling native method: Exception occurred while executing exported method getDevicePushTokenAsync on module ExpoPushTokenManager: Default FirebaseApp is not initialized in this process ...

In ./android/app/src/main/AndroidManifest.xml request a new permission as mentioned in [expo doc](https://docs.expo.dev/versions/latest/sdk/notifications/#android):

    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

After running the app, you should now see a different error message:

> getDevicePushTokenAsync [Error: Encountered an exception while calling native method: Exception occurred while executing exported method getDevicePushTokenAsync on module ExpoPushTokenManager: Failed resolution of: Lcom/google/firebase/iid/FirebaseInstanceId;]

which can be [solved by](https://stackoverflow.com/a/68320579) editing ./android/build.gradle and adding

    implementation platform('com.google.firebase:firebase-bom:28.4.2')
    implementation 'com.google.firebase:firebase-iid' // <--- this line

After killing the app and restarting you shoud see your push token in console.

🎉Congratulations your application is configured with FCM 🎉

All changes can be seen in this [commit](https://github.com/pcsikos/expo-notifications-bare-workflow/commit/57688e7958c866be4c321a5e28a41546a76e4e17)

### Other resources:

If you want to change your package name, [follow this guide](https://stackoverflow.com/a/37390022) how to do it.

[Using FCM for Push Notifications](https://docs.expo.dev/push-notifications/using-fcm/)

[Notifications](https://docs.expo.dev/versions/latest/sdk/notifications/#installation)

# Setting Up a Service Account for FCM

1. Open [APIs dashboard](https://console.cloud.google.com/apis/dashboard)
2. Create new project
3. Navigate to Credentials
4. Create Credentials -> Service Account
5. Name the account
6. Choose role 'Service Account Token Creator'
7. Done

# Generate Access Key

1. Open the service account
2. Navigate to Keys
3. Add Key -> Generate New Key -> Json
4. Store the json and use on wherever you need

🍀 Good Luck
