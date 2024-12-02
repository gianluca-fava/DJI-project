# IMPLEMENTAZIONE di MOBILE SDK 

A grandi linee viene seguito il tutorial ufficiale DJI: [Tutorial](https://developer.dji.com/doc/mobile-sdk-tutorial/en/quick-start/run-sample.html)

Vengono aggiunte alcune accortezze tratte dalla sezioen forum di DJI: [Secondo Tutorial](https://sdk-forum.dji.net/hc/en-us/articles/5983558993433-Chapter-2-The-Sample)

1. Installare Android Studio Giraffe | 2022.3.1 Patch 4: https://developer.android.com/studio/archive?hl=it
2. scaricare https://github.com/dji-sdk/Mobile-SDK-Android-V5/tree/dev-sdk-main
3. Importare in un nuovo progetto solo la cartella  android-sdk-v5-as ma tenere tutto (sample e uxdx servono come moduli, non eliminarli)
4. Usare Java 17 o superiore (File/Project Structure/SDK Location e cliccare su Gradle Settings alla fine della pagina → quindi selezionare Gradle JDK)
5. Creare una nuova applicazione su dji developer [https://developer.dji.com/](https://developer.dji.com/user/apps/#all) usare come package name “com.dji.sampleV5.aricraft”

![Screenshot 2024-11-18 alle 3.02.25 PM.png](TIROCINIO%2014fcccd555984d1eb52df0e3238f32ad/Screenshot_2024-11-18_alle_3.02.25_PM.png)

- in grandle.propreties cambiate la `AIRCRAFT_API_KEY` mettendo quella della API creata
- fare sync
- creare una nuova configurazione android app e creare un modulo nuovo con lo stesso package name dell’api
- rifare il sync project (dovrebbe apparire la vista android e si dovrebbe creare un modulo in cima alla vista)

- In gradle-wrapper.propreties cambiare distributionURL :

```
distributionUrl=https\://services.gradle.org/distributions/gradle-7.6.2-bin.zip
```

- in build.gradle (Project) aggiungere le cose mancanti:

```
// Top-level build file where you can add configuration options common to all sub-projects/modules.
apply from: rootProject.file('dependencies.gradle')

buildscript {
    repositories {
        google()
        jcenter()
        mavenCentral()
        maven { url 'https://maven.fabric.io/public' }
        maven { url 'https://plugins.gradle.org/m2/' }
        maven { url 'https://dl.bintray.com/kotlin/kotlin-eap' }
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:7.4.2'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$KOTLIN_VERSION"
    }
}

allprojects {
    repositories {
        maven {
            url REPO_MAVEN2
        }
        google()
        jcenter()
        mavenCentral()
        maven {
            url KOTLIN_MAVEN
        }
        maven { url JITPACK_MAVEN_URL }
        repositories {
            flatDir {
                dirs new File(rootProject.projectDir.getAbsolutePath() + '/libs')
            }
        }
        maven { url 'https://maven.aliyun.com/repository/public' }
        maven { url 'https://maven.aliyun.com/repository/google' }
        maven { url 'https://repo.huaweicloud.com/repository/maven'}
    }
}
```

- Nel build.gradle del module sample aggiungere le cose mancanti:

```python
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
apply plugin: 'kotlin-kapt'

android {
    compileSdkVersion Integer.parseInt(project.ANDROID_COMPILE_SDK_VERSION)

    defaultConfig {
        applicationId "com.dji.sampleV5.aircraft"
        minSdkVersion Integer.parseInt(project.ANDROID_MIN_SDK_VERSION)
        targetSdkVersion Integer.parseInt(project.ANDROID_TARGET_SDK_VERSION)
        versionCode 1
        versionName "1.0"
        manifestPlaceholders["API_KEY"] = project.AIRCRAFT_API_KEY
        manifestPlaceholders["GMAP_API_KEY"] = project.GMAP_API_KEY
        manifestPlaceholders["MAPLIBRE_TOKEN"] = project.MAPLIBRE_TOKEN
        ndk {
            abiFilters 'arm64-v8a'
        }
    }

    //配置签名信息
    signingConfigs {
        release {
            storeFile file(project.STORE_FILE)
            storePassword project.STORE_PASSWORD
            keyAlias project.KEY_ALIAS
            keyPassword project.KEY_PASSWORD
        }
    }

    buildTypes {
        release {
            minifyEnabled true
            shrinkResources false
            signingConfig signingConfigs.release
        }
        debug {
            minifyEnabled false
            shrinkResources false
        }
    }

    packagingOptions {
        // 因为mrtc库内部使用了NDK的c++_shared的编译参数
        // 与其他库重复引用了，因此选其中一个即可
        pickFirst 'lib/arm64-v8a/libc++_shared.so'
        pickFirst 'lib/armeabi-v7a/libc++_shared.so'
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    kotlinOptions {
        jvmTarget = JavaVersion.VERSION_1_8
        freeCompilerArgs += ["-Xjvm-default=all"]
    }

    //关闭lint
    lintOptions {
        checkReleaseBuilds false
        abortOnError false
    }

    packagingOptions {
        doNotStrip "*/*/libconstants.so"
        doNotStrip "*/*/libdji_innertools.so"
        doNotStrip "*/*/libdjibase.so"
        doNotStrip "*/*/libDJICSDKCommon.so"
        doNotStrip "*/*/libDJIFlySafeCore-CSDK.so"
        doNotStrip "*/*/libdjifs_jni-CSDK.so"
        doNotStrip "*/*/libDJIRegister.so"
        doNotStrip "*/*/libdjisdk_jni.so"
        doNotStrip "*/*/libDJIUpgradeCore.so"
        doNotStrip "*/*/libDJIUpgradeJNI.so"
        doNotStrip "*/*/libDJIWaypointV2Core-CSDK.so"
        doNotStrip "*/*/libdjiwpv2-CSDK.so"
        doNotStrip "*/*/libFlightRecordEngine.so"
        doNotStrip "*/*/libvideo-framing.so"
        doNotStrip "*/*/libwaes.so"
        doNotStrip "*/*/libagora-rtsa-sdk.so"
        doNotStrip "*/*/libc++.so"
        doNotStrip "*/*/libc++_shared.so"
        doNotStrip "*/*/libmrtc_28181.so"
        doNotStrip "*/*/libmrtc_agora.so"
        doNotStrip "*/*/libmrtc_core.so"
        doNotStrip "*/*/libmrtc_core_jni.so"
        doNotStrip "*/*/libmrtc_data.so"
        doNotStrip "*/*/libmrtc_log.so"
        doNotStrip "*/*/libmrtc_onvif.so"
        doNotStrip "*/*/libmrtc_rtmp.so"
        doNotStrip "*/*/libmrtc_rtsp.so"
    }
}

dependencies {
    /** <-----------------依赖MSDK--------------------> **/
    compileOnly deps.aircraftProvided
    implementation deps.aircraft

    /** <-----------------sample所需--------------------> **/
    implementation project(':uxsdk')
    implementation deps.appcompat
    implementation deps.constraintLayout
    implementation deps.aacCommon
    implementation deps.aacRuntime
    implementation deps.kotlinLib
    implementation deps.ktxCore
    implementation deps.ktxFrag
    implementation deps.ktxNavigation
    implementation deps.ktxNavigationUi
    implementation deps.recyclerview
    implementation deps.legacySupport
    implementation deps.lifecycleViewModel
    implementation deps.lifecycleLiveData
    implementation deps.leakcanary
    implementation deps.glide
    implementation deps.dynamicanimation
    implementation deps.expandedit
    implementation deps.rx3Kt
    implementation deps.dom
    kapt deps.glidecompiler
    implementation deps.lynx

}
```

- Nel file AndoridManifest.xml dentro la cartella del modulo sample aggiungere, notare alcune cose importanti:
    - package="dji.sampleV5.aircraft"
    - <activity android:name="dji.sampleV5.aircraft.DJIAircraftMainActivity" …. deve essere presente per indicare qual’è il file main.

```python
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="dji.sampleV5.aircraft">

    <!-- Sample permission requirement -->
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" android:maxSdkVersion="32" />
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
    <uses-permission android:name="android.permission.READ_MEDIA_VIDEO" />
    <uses-permission android:name="android.permission.READ_MEDIA_AUDIO" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE"/>

    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

    <!-- Google Maps -->
    <meta-data
        android:name="com.google.android.geo.API_KEY"
        android:value="${GMAP_API_KEY}" />

    <uses-feature
        android:name="android.hardware.usb.host"
        android:required="false"/>
    <uses-feature
        android:name="android.hardware.usb.accessory"
        android:required="true"/>

    <application
        android:name="dji.sampleV5.aircraft.DJIAircraftApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_main"
        android:label="@string/app_name_aircraft"
        android:supportsRtl="true"
        android:requestLegacyExternalStorage="true"
        android:theme="@android:style/Theme.NoTitleBar.Fullscreen">

        <meta-data
            android:name="com.dji.sdk.API_KEY"
            android:value="7478e6a82cc9bb0168bb8a35"/>

        <!-- Maplibre Token-->
        <meta-data
            android:name="com.dji.mapkit.maplibre.apikey"
            android:value="${MAPLIBRE_TOKEN}" />
        <activity
            android:name="dji.sampleV5.aircraft.DJIAircraftMainActivity"
            android:theme="@style/full_screen_theme"
            android:screenOrientation="landscape"
            android:configChanges="orientation|screenSize|keyboardHidden"
            android:exported="true">

            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity android:name=".UsbAttachActivity"
            android:theme="@style/translucent_theme"
            android:exported="true">
            <intent-filter>
                <action android:name="android.hardware.usb.action.USB_ACCESSORY_ATTACHED" />
            </intent-filter>

            <meta-data
                android:name="android.hardware.usb.action.USB_ACCESSORY_ATTACHED"
                android:resource="@xml/accessory_filter" />
        </activity>

        <activity android:name="dji.v5.ux.sample.showcase.defaultlayout.DefaultLayoutActivity"
            android:theme="@style/full_screen_theme"
            android:screenOrientation="landscape"
            android:configChanges="orientation|screenSize|keyboardHidden"
            android:exported="false"/>

        <activity android:name="dji.v5.ux.sample.showcase.widgetlist.WidgetsActivity"
            android:theme="@style/full_screen_theme"
            android:screenOrientation="landscape"
            android:configChanges="orientation|screenSize|keyboardHidden"
            android:exported="false"/>

        <activity android:name="dji.sampleV5.aircraft.AircraftTestingToolsActivity"
            android:theme="@style/full_screen_theme"
            android:screenOrientation="landscape"
            android:configChanges="orientation|screenSize|keyboardHidden"
            android:exported="false"/>

    </application>

</manifest>
```

- Al contrario di come scritto nella guida, in file>Project structure> Moduli non è necessari che ci siano moduli ma piuttosto questi si dovrebbero vedere in Variables.

![Screenshot 2024-11-18 alle 3.18.42 PM.png](TIROCINIO%2014fcccd555984d1eb52df0e3238f32ad/Screenshot_2024-11-18_alle_3.18.42_PM.png)

- Ora si può fare il RUN dell’applicazione (collegando fisicamente il telecomando al PC) !!!!!!!DISATTIVARE l’applicazione nativa di DJI forzandone l’uscita (altrimenti la nuova MSDK app non sarà in grado di connettersi)

[https://github.com/dji-sdk/Mobile-SDK-Android-V5/issues/401](https://github.com/dji-sdk/Mobile-SDK-Android-V5/issues/401)

https://sdk-forum.dji.net/hc/en-us/requests/120660

Per quanto riguarda l’implementazione dei vari metodi basta seguire questa guida:

[https://developer.dji.com/api-reference-v5/android-api/Components/SDKManager/DJISDKManager.html](https://developer.dji.com/api-reference-v5/android-api/Components/SDKManager/DJISDKManager.html)

Di base quello che ho fatto è stato modificare il file DJIMainActivity affinchè potessi richiamare altri metodi.

Di seguito riporto un metodo per ottenere i dati del laser e avviare i motori se legge una distanza inferiore a 3m.

```python
abstract class DJIMainActivity : AppCompatActivity() {

    val tag: String = LogUtils.getTag(this)
    private val permissionArray = arrayListOf(
        Manifest.permission.RECORD_AUDIO,
        Manifest.permission.KILL_BACKGROUND_PROCESSES,
        Manifest.permission.ACCESS_COARSE_LOCATION,
        Manifest.permission.ACCESS_FINE_LOCATION,
    )

    init {
        permissionArray.apply {
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
                add(Manifest.permission.READ_MEDIA_IMAGES)
                add(Manifest.permission.READ_MEDIA_VIDEO)
                add(Manifest.permission.READ_MEDIA_AUDIO)
            } else {
                add(Manifest.permission.READ_EXTERNAL_STORAGE)
                add(Manifest.permission.WRITE_EXTERNAL_STORAGE)
            }
        }
    }

    private val baseMainActivityVm: BaseMainActivityVm by viewModels()
    private val msdkInfoVm: MSDKInfoVm by viewModels()
    private val msdkManagerVM: MSDKManagerVM by globalViewModels()
    private val handler = Handler(Looper.getMainLooper())
    private val disposable = CompositeDisposable()

    abstract fun prepareUxActivity()

    abstract fun prepareTestingToolsActivity()

    private lateinit var aircraftAttitudeView: AircraftAttitudeView  //AGGIUNTO !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // 有一些手机从系统桌面进入的时候可能会重启main类型的activity
        // 需要校验这种情况，业界标准做法，基本所有app都需要这个
        if (!isTaskRoot && intent.hasCategory(Intent.CATEGORY_LAUNCHER) && Intent.ACTION_MAIN == intent.action) {

            finish()
            return

        }

        window.decorView.apply {
            systemUiVisibility =
                View.SYSTEM_UI_FLAG_HIDE_NAVIGATION or View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY or View.SYSTEM_UI_FLAG_FULLSCREEN or View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN
        }

        initMSDKInfoView()
        observeSDKManager()
        checkPermissionAndRequest()

        ////////////////testing////////////////

        getLaserInfo()

    }

    private fun getLaserInfo() {
        CoroutineScope(Dispatchers.Main).launch {
            while (isActive) {
                delay(10) // Aspetta X secondi

                // Recupera le informazioni del laser dal KeyManager
                val distance = KeyManager.getInstance().getValue(
                    KeyTools.createKey(CameraKey.KeyLaserMeasureInformation) //CameraKey.KeyLaserMeasureInformation, FlightControllerKey.KeyAircraftAttitude, FlightControllerKey.KeyAltitude, FlightControllerKey.KeyAircraftLocation, GimbalKey.KeyGimbalAttitude
                )
                Log.d("LASER", "Drone altitdue $distance")

                // Controlla se i dati ricevuti sono di tipo LaserMeasureInformation
                if (distance is LaserMeasureInformation) {
                    // Ottieni la distanza
                    val distance = distance.distance

                    // Se la distanza è minore di 3 metri, logga il risultato
                    if (distance < 3.0f) {
                        Log.d("LASER", "Distanza < 3m: $distance m")
                        startTakeoff()
                    } else {
                        Log.d("LASER", "Distanza > 3m: $distance m")
                    }

                } else {
                    Log.e("LASER", "Dati non validi ricevuti o tipo errato")
                }
            }
        }
    }
```