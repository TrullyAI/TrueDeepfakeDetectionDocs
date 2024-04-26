# TrueDeepfakeDetectionKotlin
 TrueDeepfakeDetection is a liveness validator especially equip to detect deepfake fraud. 

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

```

#### Groovy DSL

```groovy
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
```

### 3.- Add dependencies to the App level `build.gradle`

#### Kotlin DSL

```groovy
dependencies {
    implementation("com.github.TrullyAI:TrueDeepfakeDetectionKotlin:1.0.0")
    // Support for Java 8 features
    coreLibraryDesugaring("com.android.tools:desugar_jdk_libs:2.0.4")
}
```

#### Groovy DSL

```groovy
dependencies {
    implementation 'com.github.TrullyAI:TrueDeepfakeDetectionKotlin:1.0.0'
    // Support for Java 8 features
    coreLibraryDesugaring 'com.android.tools:desugar_jdk_libs:2.0.4'
}
```

### 4.- Add permission in manifest

```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.CAMERA" />
```

## Add it to you're project

### Listener

Add LivenessResultListener to the activity that will launch the SDK and
implement its members so you can have access to the process data.

| Method     | Description                         |
| ---------- | ----------------------------------- |
| `onResult` | Catch the results of the operation. |
| `onError`  | Catch the errors of the operation.  |

```java
class MainActivity : AppCompatActivity(), LivenessResultListener {

    override fun onResult(response: TrullyResponse) {
        Log.d("onresult", response.toString())
    }

    override fun onError(errorData: ErrorData) {
        Log.d("onError", errorData.toString())
    }
}
```

### Listeners data structure

Here you'll found the structures of the data received in each listener function

### onResult

This listener function will be called when the operation gets the Decision Maker
result.

| key         | Description                                                                                                    |
| ----------- | -------------------------------------------------------------------------------------------------------------- |
| `isAlive`   | Boolean. If true, the video is real.                                                                           |
| `selfie`    | Base64 string. An image taken during the analysis.                                                             |
| `userId`    | String. The userId you passed during configuration.                                                            |
| `tag`       | String. The tag from the process. Automatically generated when you didn't pass one through configuration prop. |
| `sessionId` | String. The id from the valid analysis.                                                                        |

```java
class MainActivity : AppCompatActivity(), LivenessResultListener {
    override fun onResult(response: TrueDeepfakeDetectionResponse) {
        Log.d("Response - MainActivity - isAlive", response.isAlive.toString())
        Log.d("Response - MainActivity - selfie", response.selfie)
        Log.d("Response - MainActivity - userId", response.userId)
        Log.d("Response - MainActivity - tag", response.tag)
        Log.d("Response - MainActivity - sessionId", response.sessionId)
    }
}
```

### onError

This listener function will be called in case of an error during the operation.

| Key         | Description                                     |
| ----------- | ----------------------------------------------- |
| `process`   | Which part of the process trigger the function. |
| `message`   | Error message                                   |
| `userID`    | The userID you passed during initialization.    |
| `timestamp` | UTC timezone. When the function was trigger.    |

### Configure

To configure the SDK you'll need to call the `init` method.

| Parameter | Description                                                       |
| --------- | ----------------------------------------------------------------- |
| `apiKey`  | You're client API_KEY. The SDK won't work without it.             |
| `config`  | Config object will pass the environment and the styles to the SDK |

```java
    TrueDeepfakeDetection.init(apiKey, config)
```

#### config object

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

Optionally you can change colors and logo. These are the default values

##### Colors

| Key               | Description                                           | Value   |
| ----------------- | ----------------------------------------------------- | ------- |
| `primaryColor`    | Will change statusBar, button, icons and links color. | #475FFF |
| `backgroundColor` | Will change button color when legal is not accepted.  | #FFFFFF |

##### Logo

We are using the url to show you the images. Please, make sure you upload the
images to your project and pass the corresponding drawable to the styles object

| Key    | Value                                                                 |
| ------ | --------------------------------------------------------------------- |
| `logo` | https://trully-api-documentation.s3.amazonaws.com/trully-sdk/logo.png |

```java
  private fun initialize() {
    val styles: TrullyStyles = TrullyStyles()

    styles.logo = ai.trully.truedeepfakedetection.R.drawable.cross
    styles.primaryColor = androidx.appcompat.R.color.error_color_material_light
    styles.backgroundColor = androidx.appcompat.R.color.button_material_dark

    styles.uiTexts.docType = Texts.INE

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

### Launch

To start the SDK you'll need to call the `start` method.

| Parameter        | Description                                                 |
| ---------------- | ----------------------------------------------------------- |
| `packageContext` | Is the context of your Application/Activity.                |
| `listener`       | Is the LivenessResultListener of your Application/activity. |

```java
    TrueDeepfakeDetection.start(packageContext, listener)
```

### Full Example

```java
class MainActivity : AppCompatActivity(), LivenessResultListener {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        findViewById<Button>(R.id.launchTrully)
            .setOnClickListener {
                initialize()
            }
    }

    override fun onResult(response: TrueDeepfakeDetectionResponse) {
        Log.d("onResult", response.toString())
    }

    override fun onError(errorData: ErrorData) {
        Log.d("onError", errorData.toString())
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
