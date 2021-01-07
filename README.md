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

    // Or comment the dependency above and uncomment the dependency below to
    // use unbundled model that depends on Google Play Services
    // implementation 'com.google.android.gms:play-services-mlkit-barcode-scanning:16.1.3'


    // CameraX
/*    implementation "androidx.camera:camera-camera2:1.0.0-SNAPSHOT"
    implementation "androidx.camera:camera-lifecycle:1.0.0-SNAPSHOT"
    implementation "androidx.camera:camera-view:1.0.0-SNAPSHOT"

  */
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

     @Nullable
     private var previewUseCase: Preview? = null

    lateinit var imageCapture: ImageCapture

    lateinit var cameraView: PreviewView
    lateinit var camera: Camera

    lateinit var cameraSelector: CameraSelector

    // if camera permission is granted 
    
    private fun startCameraMLKit() {

        //val executor: Executor = Executors.newSingleThreadExecutor()
        val executor = ContextCompat.getMainExecutor(this)


        // Listener that will help us in future to identify if camera is attached or not
        val cameraProvider = ProcessCameraProvider.getInstance(this)
        //Runnable will notify us view and changes , and second param is main Thread
        cameraProvider.addListener(Runnable {

            try {
                bindPreviewMlCameraView(cameraProvider.get())
            } catch (e: ExecutionException) {
                // No errors need to be handled for this Future.
                // This should never be reached.
            } catch (e: InterruptedException) {
            }
        }, executor)


    }

     // binding camera with view
     private fun bindPreviewMlCameraView(cameraProvider: ProcessCameraProvider) {


        previewUseCase = Preview.Builder().build()
        previewUseCase?.setSurfaceProvider(cameraView.surfaceProvider)

        imageCapture = ImageCapture.Builder().build()//for image etc

        //camera face
        cameraSelector =
            CameraSelector.Builder().requireLensFacing(CameraSelector.LENS_FACING_BACK).build()

        //val executor: Executor = Executors.newSingleThreadExecutor()
        val executor = ContextCompat.getMainExecutor(this)


        val imageAnalyzer = ImageAnalysis.Builder()
            .build()
            .also {
                it.setAnalyzer(executor, YourImageAnalyzer(this))
            }


        cameraProvider.unbindAll()//unbinding if it already exist

        camera =
            cameraProvider.bindToLifecycle(
                this,
                cameraSelector,
                previewUseCase,
                imageAnalyzer
            )


    }


     //class for observing camera view changes
    private class YourImageAnalyzer(val context:Context) : ImageAnalysis.Analyzer {

        val TAG_PROXY = "ImageProxy"

        var options: BarcodeScannerOptions = BarcodeScannerOptions.Builder()
            .setBarcodeFormats(
                Barcode.FORMAT_ALL_FORMATS
            )
            .build()

        val scanner = BarcodeScanning.getClient(options)


        @RequiresApi(Build.VERSION_CODES.KITKAT)
        @SuppressLint("UnsafeExperimentalUsageError")
        override fun analyze(imageProxy: ImageProxy) {


            try {
                val mediaImage = imageProxy.image
                if (mediaImage != null) {

                    val image =
                        InputImage.fromMediaImage(mediaImage, imageProxy.imageInfo.rotationDegrees)

                    Log.d(TAG_PROXY, "success")
                    scan(image, imageProxy)

                }
            } catch (ex: java.lang.Exception) {
                ex.message
                Log.d(TAG_PROXY, "Failure")
                imageProxy.close()
            }


        }

        public fun scan(image: InputImage, imageProxy: ImageProxy) {


            val result = scanner.process(image)
                .addOnSuccessListener { barCodes ->


                    if (barCodes.isEmpty()) {
                        Log.d(TAG_PROXY, "BarCode:Empty")
                        return@addOnSuccessListener
                    }

                    // Task completed successfully
                    // [START_EXCLUDE]
                    // [START get_barcodes]
                    for (barcode in barCodes) {
                        val bounds = barcode.boundingBox
                        val corners = barcode.cornerPoints

                        val rawValue = barcode.rawValue

                        val valueType = barcode.valueType
                        // See API reference for complete list of supported types
                        Log.d("$TAG_PROXY: TESTING: ", "Barcode detected: $rawValue")
                        Toast.makeText(context, "BarCode:$rawValue", Toast.LENGTH_LONG).show()
                    }
                }
                .addOnFailureListener {
                    Log.d("$TAG_PROXY: TESTING: ", "Failure ${it.message}")
                    imageProxy.close()
                }
                .addOnCompleteListener {
                    Log.v("$TAG_PROXY: TESTING: ", "Barcode detected: Completed")
                    imageProxy.close()

                }
        }

    }

