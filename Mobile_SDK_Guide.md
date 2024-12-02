# IMPLEMENTAZIONE di MOBILE SDK 

A grandi linee viene seguito il tutorial ufficiale DJI: [Tutorial](https://developer.dji.com/doc/mobile-sdk-tutorial/en/quick-start/run-sample.html)

Vengono aggiunte alcune accortezze tratte dalla sezioen forum di DJI: [Secondo Tutorial](https://sdk-forum.dji.net/hc/en-us/articles/5983558993433-Chapter-2-The-Sample)

1. Installare Android Studio Giraffe | 2022.3.1 Patch 4: dall'[archivio Android](https://developer.android.com/studio/archive?hl=it)
2. scaricare il progetto da GhitHub: [Mobile-SDK-Android-V5](https://github.com/dji-sdk/Mobile-SDK-Android-V5/tree/dev-sdk-main)
3. Importare in un nuovo progetto solo la cartella  android-sdk-v5-as ma tenere tutto (sample e uxdx servono come moduli, non eliminarli)
4. Usare Java 17 o superiore (File/Project Structure/SDK Location e cliccare su Gradle Settings alla fine della pagina → quindi selezionare Gradle JDK)
5. Creare una nuova applicazione su dji developer [https://developer.dji.com/](https://developer.dji.com/user/apps/#all) usare come package name “com.dji.sampleV5.aricraft” e piattaforma "Android", il nome non è rilevante.
6. In grandle.propreties cambiate la `AIRCRAFT_API_KEY` mettendo quella della API creata
7. Fare sync del progetto
8. Creare una nuova configurazione android app e creare un modulo nuovo con lo stesso package name dell’api
9. Rifare il sync project (dovrebbe apparire la vista android e si dovrebbe creare un modulo in cima alla vista)
10. In gradle-wrapper.propreties cambiare distributionURL:
```bash
distributionUrl=https\://services.gradle.org/distributions/gradle-7.6.2-bin.zip
```
11. in build.gradle (Project) aggiungere le cose mancanti:
```kotlin
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
12. Nel build.gradle del module sample aggiungere le cose mancanti:
```kotlin
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
13. Nel file AndoridManifest.xml dentro la cartella del modulo sample aggiungere, notare alcune cose importanti:
    - package="dji.sampleV5.aircraft"
    - <activity android:name="dji.sampleV5.aircraft.DJIAircraftMainActivity" …. deve essere presente per indicare qual’è il file main.
    ```kotlin
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
14. Al contrario di come scritto nella guida, in file>Project structure>Moduli non è necessario che ci siano moduli ma piuttosto questi si dovrebbero vedere in Variables.
15. Ora si può fare il RUN dell’applicazione (collegando fisicamente il telecomando al PC) 
    !!!!!!! È importantissimo DISATTIVARE l’applicazione nativa di DJI forzandone l’uscita (altrimenti la nuova MSDK app non sarà in grado di connettersi)

## POSSIBILI ERRORI:
Problema con le dipendenze: [Could not find com.mapbox.mapboxsdk:mapbox-android-accounts:0.7.0](https://github.com/dji-sdk/Mobile-SDK-Android-V5/issues/401)
[issue](https://sdk-forum.dji.net/hc/en-us/requests/120660)


## IMPLEMENTAZIONE DEI VARI METODI API
Per quanto riguarda l’implementazione dei vari metodi, la cosa migliore a mio parere è:
1. Cercare dalla [Guida DJI alle API](https://developer.dji.com/api-reference-v5/android-api/Components/SDKManager/DJISDKManager.html) il metodo che si vuole implementare, verificando a quale classe appratine e quali pacchetti necessita.
2. Verificare fisicamente tramite i tools della Mobile SDK dal telecomando, l'API che si vuole implementare, per vedere se è fattibile/eseguibile e se da i risultati voluti.
3. Andando sul sito DJI sezione community, alla pagina Mobile SDK si pososno trovare i [Tutorial](https://sdk-forum.dji.net/hc/en-us/articles/9886838527001-Table-of-contents-for-MSDK-v5-doc) e si può vedere il funzionamento della classe. Spesso c'è un diagramma a blocchi che spiega come richiamare i metodi, o comunque da un idea generale della logica che sta dietro.
4. In questa pagine spesso c'è una sezione commenti che indica se qualche metodo ha problmei di implementazione o richiede chiamate particolari. Utile per alcune API che non si capisce moltoc ome implementarle.
5. Implementare i metodi, di solito partendo dal file DJIMainActivity e richiamare i metodi.

Una buona strategia può esssere qulla di partire dai metodi che usano KeyManager e KeyTools che sono più semplici ed immediati.

Di seguito lascio la mia implemnetazione di alcuni metodi:

```kotlin
package dji.sampleV5.aircraft

//import dji.v5.ux.core.ui.hsi.AircraftAttitudeViewTEST

import android.Manifest
import android.annotation.SuppressLint
import android.content.Intent
import android.os.Build
import android.os.Bundle
import android.os.Handler
import android.os.Looper
import android.util.Log
import android.view.View
import androidx.activity.result.contract.ActivityResultContracts
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import dji.sampleV5.aircraft.models.BaseMainActivityVm
import dji.sampleV5.aircraft.models.MSDKInfoVm
import dji.sampleV5.aircraft.models.MSDKManagerVM
import dji.sampleV5.aircraft.models.globalViewModels
import dji.sampleV5.aircraft.util.Helper
import dji.sampleV5.aircraft.util.ToastUtils
import dji.sdk.keyvalue.key.BatteryKey
import dji.sdk.keyvalue.key.CameraKey
import dji.sdk.keyvalue.key.DJIKey
import dji.sdk.keyvalue.key.FlightControllerKey
import dji.sdk.keyvalue.key.GimbalKey
import dji.sdk.keyvalue.key.KeyTools
import dji.sdk.keyvalue.msdkkeyinfo.KeyCameraZoomRatios
import dji.sdk.keyvalue.value.camera.LaserMeasureInformation
import dji.sdk.keyvalue.value.common.CameraLensType
import dji.sdk.keyvalue.value.common.EmptyMsg
import dji.sdk.keyvalue.value.gimbal.GimbalResetType
import dji.v5.common.callback.CommonCallbacks
import dji.v5.common.error.IDJIError
import dji.v5.et.action
import dji.v5.et.create
import dji.v5.manager.KeyManager
import dji.v5.manager.aircraft.perception.PerceptionManager
import dji.v5.utils.common.LogUtils
import dji.v5.utils.common.PermissionUtil
import dji.v5.utils.common.StringUtils
import dji.v5.ux.core.ui.hsi.AircraftAttitudeView.*
import io.reactivex.rxjava3.disposables.CompositeDisposable
import kotlinx.android.synthetic.main.activity_main.*
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.delay
import kotlinx.coroutines.isActive
import kotlinx.coroutines.launch


/**
 * Class Description
 *
 * @author Hoker
 * @date 2022/2/10
 *
 * Copyright (c) 2022, DJI All Rights Reserved.
 */
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

        ////////testing

        //startDataCollection(1000)

        //monitorLaserDistance()

        //startVoltageMonitoring()

        //performGimbalRotation(GimbalResetType.SELFIE)

        //getRadarInfo()

    }

    ////////////////

    fun startDataCollection(delayTime: Long) {
        // Avvia una coroutine sul thread principale per raccogliere e salvare i dati periodicamente
        CoroutineScope(Dispatchers.Main).launch {
            while (isActive) {

                delay(delayTime) // 5 secondi

                // Ottieni i dati dal drone tramite il KeyManager
                val attitudeData = getAttitudeData()
                val altitudeData = getAltitudeData()
                val batteryData = getBattryData()
                val locationData = getLocationData()
                val gimbalData = getGimbalData()
                val laserData = getLaserData()

                // Stampa i dati nel log
                logData(attitudeData, altitudeData, batteryData, locationData, gimbalData, laserData)
            }
        }
    }

    // Funzione per ottenere i dati sull'orientamento del drone (attitude)
    private fun getAttitudeData(): Any? {
        return KeyManager.getInstance().getValue(
            KeyTools.createKey(FlightControllerKey.KeyAircraftAttitude)
        )
    }

    // Funzione per ottenere i dati sull'altitudine (se il drone ha i motori spenti si riazzera continaumante)
    private fun getAltitudeData(): Any? {
        return KeyManager.getInstance().getValue(
            KeyTools.createKey(FlightControllerKey.KeyAltitude)
        )
    }

    // Funzione per ottenere i dati sul voltaggio della batteria
    private fun getBattryData(): Any? {
        return KeyManager.getInstance().getValue(
            KeyTools.createKey(BatteryKey.KeyVoltage)
        )
    }

    // Funzione per ottenere i dati sulla posizione dell'aeromobile
    private fun getLocationData(): Any? {
        return KeyManager.getInstance().getValue(
            KeyTools.createKey(FlightControllerKey.KeyAircraftLocation)
        )
    }

    // Funzione per ottenere i dati sull'orientamento del gimbal
    private fun getGimbalData(): Any? {
        return KeyManager.getInstance().getValue(
            KeyTools.createKey(GimbalKey.KeyGimbalAttitude)
        )
    }

    // Funzione per ottenere i dati sul laser
    private fun getLaserData(): Any? {
        return KeyManager.getInstance().getValue(
            KeyTools.createKey(CameraKey.KeyLaserMeasureInformation)
        )
    }

    private fun logData(
        laser: Any?,
        attitude: Any?,
        battery: Any?,
        altitude: Any?,
        location: Any?,
        gimbal: Any?
    ) {
        Log.d("LOG DATA", "log data:\nLASER: $laser\nATTITUDE: $attitude\nBATTERY: $battery\nALTITUDE: $altitude\nLOCATION: $location\nGIMBAL: $gimbal")
    }



    // Funzione per raccogliere e monitorare le informazioni sul laser
    private fun monitorLaserDistance() {
        CoroutineScope(Dispatchers.Main).launch {
            // La coroutine continua a funzionare finché l'app è attiva
            while (isActive) {
                delay(1000) // Aspetta 1 secondo tra ogni lettura del laser

                // Recupera le informazioni del laser dal KeyManager
                val laserInfo = getLaserData()

                // Se i dati sono del tipo corretto, gestiscili
                if (laserInfo is LaserMeasureInformation) {
                    handleLaserDistance(laserInfo.distance)
                } else {
                    // Se i dati non sono validi o il tipo è errato, logga un errore
                    Log.e("TAKEOFF", "Invalid data or wrong type received")
                }
            }
        }
    }

    // Funzione per gestire la distanza del laser
    private fun handleLaserDistance(distance: Double) {
        // Se la distanza è minore di 3 metri, inizia il decollo
        if (distance < 3.0f) {
            Log.d("TAKEOFF", "Distance < 3m: $distance meters")
            initiateTakeoff()
        } else {
            // Se la distanza è maggiore di 3 metri, logga il valore
            Log.d("TAKEOFF", "Distance > 3m: $distance meters")
        }
    }

    fun initiateTakeoff() {
        Log.d("TAKEOFF", "Takeoff")

        val callback = object : CommonCallbacks.CompletionCallbackWithParam<EmptyMsg> {
            override fun onSuccess(t: EmptyMsg?) {
                Log.d("TAKEOFF", "start takeOff onSuccess.")
            }

            override fun onFailure(error: IDJIError) {
                Log.e("TAKEOFF", "start takeOff onFailure,$error")
                var e = error.description()
                if (e == "null") e = "unknown"
            }
        }

        FlightControllerKey.KeyStartTakeoff.create().action({
            callback.onSuccess(it)
        }, { e: IDJIError ->
            callback.onFailure(e)
        })
    }


    // Funzione per monitorare la tensione della batteria
    fun startVoltageMonitoring() {
        CoroutineScope(Dispatchers.Main).launch {
            while (isActive) {
                // Ottieni il dato relativo alla batteria
                val batteryVoltage = getBattryData()

                Log.d("BEEPER", "Tipo di batteryVoltage: $batteryVoltage")


                // Se è una stringa, converti in numero
                if (batteryVoltage is String) {
                    val voltage = batteryVoltage.toIntOrNull()
                    if (voltage != null && voltage < 2100) {
                        performBeep()
                    }
                } else if (batteryVoltage is Int) {
                    // Se 'batteryVoltage' è già un Int, controlla se è inferiore a 2100
                    if (batteryVoltage < 2100) {
                        performBeep()
                    }
                }

                // Ritardo per evitare chiamate troppo frequenti
                delay(5000) // ogni 5 secondi
            }
        }
    }

    // Funzione per eseguire il beep sul controllore
    fun performBeep(){
        CoroutineScope(Dispatchers.Main).launch {
            while (isActive) {

                // Abilita il beep sugli ESC del drone
                KeyManager.getInstance().setValue(
                    KeyTools.createKey(FlightControllerKey.KeyESCBeepEnabled),
                    true,
                    object : CommonCallbacks.CompletionCallback {
                        override fun onSuccess() {
                            Log.d("BEEPER", "ESC Beep abilitato correttamente.")
                        }
                        override fun onFailure(error: IDJIError) {
                            Log.e("BEEPER", "Errore nell'abilitare ESC Beep")
                        }
                    }
                )
            }
        }
    }


    // Funzione per eseguire la rotazione del gimbal in base al tipo di reset
    fun performGimbalRotation(resetType: GimbalResetType){
        CoroutineScope(Dispatchers.Main).launch {
            while (isActive) {
                delay(10000)

                // Esegui l'azione di reset sul gimbal
                val radar = KeyManager.getInstance().performAction(
                    KeyTools.createKey(GimbalKey.KeyGimbalReset),
                    resetType,  // Passa il tipo di reset
                    object  : CommonCallbacks.CompletionCallbackWithParam<EmptyMsg>{
                        override fun onSuccess(p0: EmptyMsg?) {
                            Log.d("GIMBAL", "Gimbal ruotato in modalità $resetType")
                        }

                        override fun onFailure(p0: IDJIError) {
                            Log.e("GIMBAL", "Errore nella rotazione gimbal")
                        }

                    }
                )
            }
        }
    }

    //funzione per prendere la distanza con i sensori,m da testare in volo
    fun getRadarInfo(){
        CoroutineScope(Dispatchers.Main).launch {
            while (isActive) {
                delay(5000)
                val radar = PerceptionManager.getInstance().radarManager.addRadarInformationListener{ value ->
                    Log.d("RADAR", "radarObstacleDataListener: $value")
                }
            }
        }
    }


    /////////////////////////////// di seguito tutto il resto come è di base
```




