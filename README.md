# TrueDeepfakeDetectionKotlin

TrueDeepfakeDetection is a liveness validator especially equip to detect
deepfake fraud.

## Add TrueDeepfakeDetection repository and dependencies

### 1.- Add jitpack as repository store in `settings.gradle`

#### Kotlin DSL

```groovy
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)

    repositories {
        google()
        mavenCentral()

        maven {
            url = uri("https://jitpack.io")
        }
    }
}
```

#### Groovy DSL

```groovy
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)

    repositories {
        google()
        mavenCentral()

        maven {
            url 'https://jitpack.io'
        }
    }
}
```

### 2.- Jetpack Compose on your App level `build.gradle`

Enable Jetpack Compose by adding the following to the android section

#### Kotlin DSL

```groovy
android {
    compileOptions {
        // Support for Java 8 features
        isCoreLibraryDesugaringEnabled = true
        sourceCompatibility = JavaVersion.VERSION_1_8
        targetCompatibility = JavaVersion.VERSION_1_8
    }

    buildFeatures {
        compose = true
    }

    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.1"
    }
}

```

#### Groovy DSL

```groovy
android {
    compileOptions {
        // Support for Java 8 features
        coreLibraryDesugaringEnabled true
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    buildFeatures {
        compose true
    }

    composeOptions {
    kotlinCompilerExtensionVersion '1.5.1'
    }
}
```

### 3.a- Add dependencies to the App level `build.gradle`

#### Kotlin DSL

```groovy
dependencies {
    implementation("com.github.TrullyAI:TrueDeepfakeDetectionKotlin:version")
    // Support for Java 8 features
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:1.1.5")
}
```

#### Groovy DSL

```groovy
dependencies {
    implementation 'com.github.TrullyAI:TrueDeepfakeDetectionKotlin:version'
    // Support for Java 8 features
    coreLibraryDesugaring 'com.android.tools:desugar_jdk_libs:1.1.5'
}
```

### 3.b- Add dependencies using the libraries system

#### `libs.versions.toml` file

```groovy
[versions]
liveness = "version"
desugaring = "1.1.5"

[libraries]
liveness = { group = "com.github.TrullyAI", name = "TrueDeepfakeDetectionKotlin", version.ref = "liveness" }
desugaring-library = { group = "com.android.tools", name = "desugar_jdk_libs", version.ref = "desugaring" }
```

#### App level `build.gradle`

```groovy
dependencies {
    implementation(libs.liveness)
    coreLibraryDesugaring(libs.desugaring.library)
}
```

### 4.- Add permission in manifest

```xml
    <uses-feature android:name="android.hardware.camera" android:required="true" />

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.CAMERA" />
```

## Add it to you're project

### Add Listeners

Add LivenessResultListener to the activity that will launch the SDK and
implement its members so you can have access to the process data.

| Method     | Description                         |
| ---------- | ----------------------------------- |
| `onResult` | Catch the results of the operation. |
| `onError`  | Catch the errors of the operation.  |

#### Example

```java
class MainActivity : AppCompatActivity(), LivenessResultListener {

    override fun onResult(response: TrullyResponse) {
        Log.d("onresult", response.toString())
    }

    override fun onError(error: ErrorData) {
        Log.d("onError", error.toString())
    }
}
```

### Configure

To configure the SDK you'll need to call the `init` method.

| Parameter | Description                                                       |
| --------- | ----------------------------------------------------------------- |
| `apiKey`  | You're client API_KEY. The SDK won't work without it.             |
| `config`  | Config object will pass the environment and the styles to the SDK |

#### Example

```java
    val config = TrullyConfig(environment = Environment.DEBUG, userID = "YOUR_ID_FOR_THE_PROCESS")
    //* For production environments use `Environment.RELEASE`. Required
    //* userID will identify the user completing the process. Required

    TrueDeepfakeDetection.init(apiKey = "YOUR_API_KEY", config = config)
```

### Launch

To start the SDK you'll need to call the `start` method.

| Parameter        | Description                                                 |
| ---------------- | ----------------------------------------------------------- |
| `packageContext` | Is the context of your Application/Activity.                |
| `listener`       | Is the LivenessResultListener of your Application/activity. |

#### Example

```java
    TrueDeepfakeDetection.start(packageContext = this, listener = this)
```

### Complete Example with default styles

```java
import ai.trully.truedeepfakedetection.TrueDeepfakeDetection
import ai.trully.truedeepfakedetection.configurations.TrullyConfig
import ai.trully.truedeepfakedetection.models.Environment
import ai.trully.truedeepfakedetection.models.ErrorData
import ai.trully.truedeepfakedetection.models.TrueDeepfakeDetectionResponse
import ai.trully.truedeepfakedetection.protocols.listeners.LivenessResultListener

class MainActivity : AppCompatActivity(), LivenessResultListener {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        initialize()
    }

    override fun onResult(response: TrueDeepfakeDetectionResponse) {
        Log.d("onResult", response.toString())
    }

    override fun onError(error: ErrorData) {
        Log.d("onError", error.toString())
    }

    private fun initialize() {
       //Set SDK configuration
       val config = TrullyConfig(environment = Environment.DEBUG, userID = "YOUR_ID_FOR_THE_PROCESS")
       //* For production environments use `Environment.RELEASE`. Required
       //* userID will identify the user completing the process. Required

        //Initialize SDK
        TrueDeepfakeDetection.init(apiKey = "YOUR_API_KEY", config = config)

        //Run SDK
        TrueDeepfakeDetection.start(packageContext = this, listener = this)
    }
}
```

### Listeners data structure

Here you'll found the structures of the data received in each listener function

### onResult

This listener function will be called when the operation gets the Decision Maker
result.

| key          | Description                                                                                                    |
| ------------ | -------------------------------------------------------------------------------------------------------------- |
| `isAlive`    | Boolean. If true, the video is real.                                                                           |
| `selfie`     | Base64 string. An image taken during the analysis.                                                             |
| `userId`     | String. The userId you passed during configuration.                                                            |
| `tag`        | String. The tag from the process. Automatically generated when you didn't pass one through configuration prop. |
| `sessionId`  | String. The id from the valid analysis.                                                                        |
| `DMResponse` | You'll find more details in [Decision Maker API Docs](https://trully.readme.io/reference/decisionmakerpredict) |

#### Example

```java
class MainActivity : AppCompatActivity(), LivenessResultListener {
    override fun onResult(response: TrueDeepfakeDetectionResponse) {
        Log.d("Response - isAlive", response.isAlive.toString())
        Log.d("Response - selfie", response.selfie)
        Log.d("Response - userId", response.userId)
        Log.d("Response - tag", response.tag)
        Log.d("Response - sessionId", response.sessionId)
        Log.d("Response - DM", response.DMResponse.toString())
    }
}
```

### DMResponse data

| Key            | Description                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------- |
| `request_id`   | String. The id of the DM Response                                                           |
| `label`        | String. The label generate by the Decision Maker for the user who has completed the process |
|                | No Threat - low risk user. Review - medium risk user. Potential Threat - high risk user     |
| `reason`       | Array. Contains the reasons behind the decision                                             |
| `request_date` | String. UTC timezone date for the request                                                   |

### onError

This listener function will be called in case of an error during the operation.

| Key         | Description                                     |
| ----------- | ----------------------------------------------- |
| `process`   | Which part of the process trigger the function. |
| `message`   | Error message                                   |
| `userID`    | The userID you passed during initialization.    |
| `timestamp` | UTC timezone. When the function was trigger.    |

#### Example

```java
    override fun onError(error: ErrorData) {
        Log.d("onError", error.toString())
    }
```

##### Process Table

| Name                  | Description                                 |
| --------------------- | ------------------------------------------- |
| `GETTING_SESSION_ID`  | HTTP error getting sessionId.               |
| `GETTING_CREDENTIALS` | HTTP error authenticating session.          |
| `GETTING_LIVENESS`    | HTTP error processing image.                |
| `GETTING_DM_RESPONSE` | HTTP error getting Decision Maker response. |

#### Personalization

The config object will allow to configure environment execution, add or delete
view from the process and change the styles. Also, for the process to work you
need to pass a userID. The config object will let you do that.

| Parameter      | Description                                                                                                         |
| -------------- | ------------------------------------------------------------------------------------------------------------------- |
| `environment`  | Environment.DEBUG for development. Environment.RELEASE for production. Mandatory                                    |
| `userID`       | Will allow you to link the process to an ID generate by you for better track of each process. Mandatory             |
| `tag`          | Valid uuid string. If you do not provide it, one will automatically generated.                                      |
|                | One analysis could run several process. Each one will be available with this tag                                    |
| `showIdView`   | Boolean. Set it to true if you want to ask your client to show their id while running the validation. Default false |
| `showExitView` | Boolean. If true the process will show a thank you final view. Default false. Shows only an icon check              |
| `styles`       | Styles object that will allow you to config color, logo and texts. Optional                                         |

#### Changing styles

Optionally you can change colors and images. These are the default values

##### Colors

| Key               | Description                                           | Value   |
| ----------------- | ----------------------------------------------------- | ------- |
| `primaryColor`    | Will change statusBar, button, icons and links color. | #475FFF |
| `backgroundColor` | Will change button color when legal is not accepted.  | #FFFFFF |

#### Example

```java
  private fun initialize() {
    val styles: TrullyStyles = TrullyStyles()

    styles.primaryColor = androidx.appcompat.R.color.error_color_material_light
    styles.backgroundColor = androidx.appcompat.R.color.button_material_dark

    //Set SDK configuration
    val config = TrullyConfig(environment = Environment.DEBUG, userID = "YOUR_ID_FOR_THE_PROCESS", tag = "VALID_UUID_STRING" , showIdView = true, showExitView = false, style = styles)
    //* For production environments use `Environment.RELEASE`.
    //* We recommend using named arguments so the order doesn't matter. If you're not using them, this example shows the order you should pass the arguments

    //Initialize SDK
    TrueDeepfakeDetection.init(apiKey = "YOUR_API_KEY", config = config)

    //Run SDK
    TrueDeepfakeDetection.start(packageContext = this, listener = this)
}
```

##### Images

We are using the url to show you the images. Please, make sure you upload the
images to your project and pass the corresponding drawable to the styles object

| Key        | Value                                                                                 |
| ---------- | ------------------------------------------------------------------------------------- |
| `logo`     | https://trully-api-documentation.s3.amazonaws.com/trully-sdk/logo.png                 |
| `light`    | https://trully-api-documentation.s3.amazonaws.com/trully-sdk/luzIcon.svg              |
| `cross`    | https://trully-api-documentation.s3.amazonaws.com/trully-sdk/retirarElementosIcon.svg |
| `showId`   | https://trully-api-documentation.s3.amazonaws.com/trully-sdk/pruebavida.svg           |
| `check`    | https://trully-api-documentation.s3.amazonaws.com/trully-sdk/icon-check.svg           |
| `timeout`  | https://trully-api-documentation.s3.amazonaws.com/trully-sdk/rostrofail.svg           |
| `noCamera` | https://trully-api-documentation.s3.amazonaws.com/trully-sdk/cameraDenied-1.svg       |
| `error`    | https://trully-api-documentation.s3.amazonaws.com/trully-sdk/timeout.svg              |

#### Example

```java
  private fun initialize() {
    val styles: TrullyStyles = TrullyStyles()

    styles.logo = ai.trully.truedeepfakedetection.R.drawable.logo
    styles.light = ai.trully.truedeepfakedetection.R.drawable.light
    styles.cross = ai.trully.truedeepfakedetection.R.drawable.cross
    styles.showId = ai.trully.truedeepfakedetection.R.drawable.pruebavida
    styles.check = ai.trully.truedeepfakedetection.R.drawable.check_circulo
    styles.timeout = ai.trully.truedeepfakedetection.R.drawable.rostrofail
    styles.noCamera = ai.trully.truedeepfakedetection.R.drawable.camara
    styles.error = ai.trully.truedeepfakedetection.R.drawable.timeout

    //Set SDK configuration
    val config = TrullyConfig(environment = Environment.DEBUG, userID = "YOUR_ID_FOR_THE_PROCESS", tag = "VALID_UUID_STRING" , showIdView = true, showExitView = false, style = styles)
    //* For production environments use `Environment.RELEASE`.
    //* We recommend using named arguments so the order doesn't matter. If you're not using them, this example shows the order you should pass the arguments

    //Initialize SDK
    TrueDeepfakeDetection.init(apiKey = "YOUR_API_KEY", config = config)

    //Run SDK
    TrueDeepfakeDetection.start(packageContext = this, listener = this)
}
```

### Full Example

```java
import ai.trully.truedeepfakedetection.TrueDeepfakeDetection
import ai.trully.truedeepfakedetection.configurations.TrullyConfig
import ai.trully.truedeepfakedetection.configurations.TrullyStyles
import ai.trully.truedeepfakedetection.models.Environment
import ai.trully.truedeepfakedetection.models.ErrorData
import ai.trully.truedeepfakedetection.models.TrueDeepfakeDetectionResponse
import ai.trully.truedeepfakedetection.protocols.listeners.LivenessResultListener

class MainActivity : AppCompatActivity(), LivenessResultListener {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        initialize()
    }

    override fun onResult(response: TrueDeepfakeDetectionResponse) {
        Log.d("onResult", response.toString())
    }

    override fun onError(error: ErrorData) {
        Log.d("onError", error.toString())
    }

    private fun initialize() {
       val styles = TrullyStyles()
       styles.logo = ai.trully.truedeepfakedetection.R.drawable.cross
       styles.primaryColor = androidx.appcompat.R.color.error_color_material_light
       styles.backgroundColor = androidx.appcompat.R.color.button_material_dark

       val config = TrullyConfig(environment = Environment.DEBUG, userID = "YOUR_ID_FOR_THE_PROCESS", tag = UUID.randomUUID().toString() , showIdView = true, showExitView = false, style = styles)

        TrueDeepfakeDetection.init(apiKey = "YOUR_API_KEY", config = config)
        TrueDeepfakeDetection.start(packageContext = this, listener = this)
    }
}
```

## How to know the app size

The size that you are seeing is not the size that your end users will get. The
size that you can see includes support for four architectures (arm64-v8a,
armeabi-v7a, x86, and x86_64). But one device has one architecture, not four.
You can check the size of the app for one architecture creating an .aab for a
specific architecture by following these instructions

### 1.- Configure the desired architecture build on your app level `build.gradle`

#### Kotlin DSL

```groovy
android {
    defaultConfig {
        ...
        ndk {
            abiFilters 'arm64-v8a'
        }
    }
}
```

#### Groovy DSL

```groovy
android {
    defaultConfig {
        ...
        ndk {
            abiFilters("arm64-v8a")
        }
    }
}
```

### 2.- Generate a signed App Bundle

In the Menu of Android Studio: Build -> Generate Signed Bundle / APK. Then
choose the Android App Bundle option. Follow the assistance instructions and the
result should be a single architecture bundle with the size your app will take
on a user device

⚠️ Make sure to delete these configuration before creating the final package
