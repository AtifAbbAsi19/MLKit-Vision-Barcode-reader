# Add ML-Kit/Vision SDK to your app

ML-Kit SDK is a-service that provides several features for building powerful mobile apps. 
## Getting Started

### Prerequisites

* A device running Android 5.0  or newer, and Google Play services 11.8.0 or higher
* [Android Studio](https://developer.android.com/studio/index.html) - The latest version of Android Studio

### Add the SDK

If you would like to integrate the Firebase libraries into one of your own projects, you need to perform a few basic tasks to prepare your Android Studio project. You may have already done this as part of adding ML-Kit to your app.

First, add rules to your root-level build.gradle file, to include the google-services plugin and the Google's Maven repository:

```
buildscript {
    // ...
    dependencies {
        // ...
        classpath 'com.google.gms:google-services:3.2.0' // google-services plugin
    }
}

allprojects {
    // ...
    repositories {
        // ...
        maven {
            url "https://maven.google.com" // Google's Maven repository
        }
    }
}

```
Then, in your module Gradle file (usually the app/build.gradle), add the apply plugin line at the bottom of the file to enable the Gradle plugin:

```
apply plugin: 'com.android.application'

android {
  // ...
}

dependencies {
  // ...
  
  // Barcode model
    implementation 'com.google.mlkit:barcode-scanning:16.1.0'
    
    //Camera X
    implementation 'androidx.camera:camera-core:1.0.0-rc01'
    implementation 'androidx.camera:camera-camera2:1.0.0-rc01'
    implementation "androidx.camera:camera-view:1.0.0-alpha20"
    implementation "androidx.camera:camera-lifecycle:1.0.0-rc01"
  
  // Getting a "Could not find" error? Make sure you have
  // added the Google maven respository to your root build.gradle
}

// ADD THIS AT THE BOTTOM
apply plugin: 'com.google.gms.google-services'
```


Then, in your Manifest.xml  file (usually the app/Manifest.xml), add the Camera & Storage Permissions.


    <!-- CameraX libraries require minSdkVersion 21, while this quickstart app
    supports low to 16. Needs to use overrideLibrary to make the merger tool
    ignore this conflict and import the libraries while keeping the app's lower
    minSdkVersion value. In code, will check SDK version, before calling CameraX
    APIs. -->
    <uses-sdk
        tools:overrideLibrary="
          androidx.camera.camera2, androidx.camera.core,
          androidx.camera.view, androidx.camera.lifecycle" />

<!--    <uses-feature android:name="android.hardware.camera"/>-->
    <!--MLKIT/Vision API &  CameraX for barcode scanner  -->
<!--    <uses-feature android:name="android.hardware.camera.autofocus" />-->
                <uses-feature android:name="android.hardware.camera.any" />


      <Application>
  
          <meta-data
            android:name="com.google.android.gms.version"
            android:value="@integer/google_play_services_version" />


        <!-- Optional: Add it to automatically download ML model to device after
          your app is installed.-->
        <meta-data
            android:name="com.google.mlkit.vision.DEPENDENCIES"
            android:value="barcode"/>
<!--            android:value="barcode,face,ocr,ica"/>-->
  
       </Application



