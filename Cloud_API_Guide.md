# IMPLEMENTAZIONE di CLOUD API

Creare l'account DJI Developer

Creare una nuova APP nell’account DJI Developer e ottenuto le chiavi API:  https://developer.dji.com/user/apps/

## IMPLEMENTAZIONE CON DOCKER

1. **Scaricare il framework**: scaricare il file https://terra-sz-hc1pro-cloudapi.oss-cn-shenzhen.aliyuncs.com/c0af9fe0d7eb4f35a8fe5b695e4d0b96/docker/cloud_api_sample_docker.zip importarlo in un nuovo progetto intelliJ IDEA
2. **Installare docker-compose**: (io ho installato la versione docker desktop esterna a intelliJ)
    
    ```bash
    https://github.com/docker/compose/releases #installare la versione corretta
    
    sudo mv ~/Downloads/docker-compose-linux-x86_64 /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose
    /usr/local/bin/docker-compose --version
    ```
    
3. **Cambiare le variabili d’ambiente prendendo i valori da DJI developer**: andando in source/nginx/front_page/src/api/http,modificare la configurazione del front-end nel file "config.ts" ed inserire le chiavi create all'inizio (appId, appKey, appLicense)
    
    ```bash
    // license
    
    appId: '******', // You need to go to the development website to apply.
    appKey: '******', // You need to go to the development website to apply.
    appLicense: '******', // You need to go to the development website to apply.
    
    // http
    
    baseURL: 'http://192.168.0.0:6789/', // This url must end with "/". Examp,le: 'http://192.168.1.1:6789/'
    websocketURL: 'ws://192.168.0.0:6789/api/v1/ws', // Example: 'ws://192.168.1.1:6789/api/v1/ws'
    
    #Nel caso in cui si volgia la livestream, cerare un profilo su agora e aggioranre anche i campi in fondo: agoraAPPID, agoraToken, agoraChannel
    ```
    
5. **Installare i pacchetti**: se i pacchetti richiesti non sono già installati, procedere come segue
    
    ```bash
    #nodeJS va bene una versione pari o successiva alla 17.8.0 (nel mio caso: node -v v20.17.0    npm -v 10.8.2)
    
    sudo apt remove nodejs
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
    
    #riavviare il terminale 
    
    source ~/.bashrc
    nvm install 17.8.0
    
    #se si ottiene un errore tipo:
    Warning: Failed to create the file 
    Warning: /home/sauronnikko/.nvm/.cache/bin/node-v12.16.3-linux-x64/node-v12.16.
    Warning: 3-linux-x64.tar.xz: Permission denied
    #eseguire:
    sudo snap remove curl
    sudo apt install curl
    
    #quindi rieseguire 
    sudo apt install curl
    
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
    nvm install 17.8.0
    
    nvm use 17.8.0
    node -v
    
    ```
    
    **- Installazione Java**
    
    ```bash
    #https://www.openlogic.com/openjdk-downloads ho istallato jdk 17.0.4
    
    #estraggo
    
    tar -xzf openjdk-17_linux-x64_bin.tar.gz
    
    #sposto la cartella
    
    sudo mkdir -p /usr/lib/jvm
    sudo mv openlogic-openjdk-21.0.4+7-linux-x64 /usr/lib/jvm/
    
    #configura variabili d'ambiente
    
    export JAVA_HOME=/usr/lib/jvm/openlogic-openjdk-21.0.4+7-linux-x64
    export PATH=$JAVA_HOME/bin:$PATH
    
    java -version
    ```
    
    **- Installazione EMQX**
    
    ```bash
    installare la versione deb corretta da: https://github.com/emqx/emqx/releases
    
    sudo dpkg -i emqx-5.8.0-ubuntu24.04-amd64.deb
    sudo apt-get install -f
    sudo systemctl start emqx
    sudo systemctl status emqx
    ```
    
    **- Installazione MySQL**
    
    ```bash
    sudo apt update
    
    sudo apt install mysql-server -y
    mysql --version
    
    sudo mysql -u root
    source cloud_api_sample/source/backend_service/sql/cloud_sample.sql
    ```
    
    **- Installazione Redis**
    
    ```bash
    dpkg -l | grep redis
    
    sudo apt-get remove --purge redis
    sudo apt-get autoremove
    
    sudo apt-get install redis
    
    ls /lib/systemd/system/redis*
    
    sudo apt-get remove --purge redis
    sudo apt-get update
    sudo apt-get install redis-server
    
    sudo systemctl enable redis-server
    sudo systemctl start redis-server
    
    sudo systemctl status redis-server
    ```
    
6. **Cambiare i campi**: andando su source/backend_service/src/main/resources, modificare la configurazione backend nel file "application.yml".
Di seguito il mio file application.yml con le parti da cambiare:
    
    ```bash
    .......
    
    mqtt:
      # @see com.dji.sample.component.mqtt.model.MqttUseEnum
      # BASIC parameters are required.
      BASIC:
        protocol: MQTT # @see com.dji.sample.component.mqtt.model.MqttProtocolEnum
        host: 192.168.0.0 #enter your public IP (the one found in your pc's network settings)
        port: 1883
        username: JavaServer
        password: 123456
        client-id: 123456
        # If the protocol is ws/wss, this value is required.
        path:
      DRC:
        protocol: WS # @see com.dji.sample.component.mqtt.model.MqttProtocolEnum
        host: 192.168.0.0 #enter your public IP (the one found in your pc's network settings)
        port: 8083
        path: /mqtt
        username: JavaServer
        password: 123456
    
    .......
    
    # Tutorial: https://www.alibabacloud.com/help/en/object-storage-service/latest/use-a-temporary-credential-provided-by-sts-to-access-oss
    
    #oss:
    #  enable: false
    #  provider: ALIYUN # @see com.dji.sample.component.OssConfiguration.model.enums.OssTypeEnum
    #  endpoint: https://oss-cn-hangzhou.aliyuncs.com
    #  access-key: Please enter your access key.
    #  secret-key: Please enter your secret key.
    #  expire: 3600
    #  region: Please enter your oss region. # cn-hangzhou
    #  role-session-name: cloudApi
    #  role-arn: Please enter your role arn. # acs:ram::123456789:role/stsrole
    #  bucket: Please enter your bucket name.
    #  object-dir-prefix: Please enter a folder name.
    
    oss:
      enable: true
      provider: aws
      endpoint: https://s3.eu-central-1.amazonaws.com #insert correct endpoint (change region basically)
      access-key: ******* #(for those create a new IAM account)
      secret-key: *******
      expire: 3600
      region: eu-central-1
      role-session-name: cloudApi
      role-arn: arn:aws:iam::****** #I copied the LambdaToDynamoDBAndS3 role (IAM/roles/LambdaToDynamoDBAndS3/ARN in aws)
      bucket: cloudapi-bucket-uniud #create a new bucket S3
      object-dir-prefix: wayline
    
    #oss:
    #  enable: true
    #  provider: minio
    #  endpoint: http://192.168.1.1:9000
    #  access-key: minioadmin
    #  secret-key: minioadmin
    #  bucket: cloud-bucket
    #  expire: 3600
    #  region: us-east-1
    #  object-dir-prefix: wayline
    
    logging:
      level:
        com.dji: debug
      file:
        name: logs/cloud-api-sample.log
    
    ntp:
      server:
        host: Google.mzr.me
    
    # To create a license for an application: https://developer.dji.com/user/apps/#all
    cloud-api:
      app:
        id: ******** #copy them from dji dev
        key: ********
        license: ********
        
    livestream:
      url:
        # It is recommended to use a program to create Token. https://github.com/AgoraIO/Tools/blob/master/DynamicKey/AgoraDynamicKey/java/src/main/java/io/agora/media/RtcTokenBuilder2.java
        agora:
          channel: 1234
          token: ******** #create an account on agora (only if you need livestream)
          uid:  654321
    
        # RTMP  Note: This IP is the address of the streaming server. If you want to see livestream on web page, you need to convert the RTMP stream to WebRTC stream.
        rtmp:
          url: Please enter the rtmp access address.  # Example: 'rtmp://192.168.1.1/live/'
        rtsp:
          username: Please enter the username.
          password: Please enter the password.
          port: 8554
    
        # GB28181 Note:If you don't know what these parameters mean, you can go to Pilot2 and select the GB28181 page in the cloud platform. Where the parameters same as these parameters.
        gb28181:
          serverIP: Please enter the server ip.
          serverPort: 1234
          serverID: Please enter the server id.
          agentID: Please enter the agent id.
          agentPassword: Please enter the agent password.
          localPort: 1234
          channel: Please enter the channel.
    
        # Webrtc: Only supports using whip standard
        whip:
          url: Please enter the rtmp access address. #  Example：http://192.168.1.1:1985/rtc/v1/whip/?app=live&stream=
    ```
    
7. **Esecuzione**: caricamento dell’immagine Docker, aggiornamento della front-end e back-end e creazione ed avvio del container
    
    ```bash
    sudo docker load < cloud_api_sample_docker_v1.10.0.tar
    
    sudo apt install curl
    
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
    
     # Build the front-end image file.
     ./update_front.sh
     (se non campare la folder node_mpodules meglio eseguire manualmente npm install spostandosi nella directory cloud_api_sample/source/nginx/front_page e rieseguire il file)
     
     # Build the back-end image file.
     ./update_backend.sh
      
     #creazione container
     sudo docker-compose up -d #questo equivale a d eseguire manualemnte il docker-compose.yml
     
     #Io eseguo sempre manualmente anche il docker file (update_front.sh + cloud_api_sample/source/nginx/Dockerfile e update_backend.sh + cloud_api_sample/docker-compose.yml) 
    ```
    

## ALCUNI POSSIBILI ERRORI:

Di seguito vengono riportati alcuni degli errori più comuni che si possono incontrare nel deployment:

```bash
ERRORE:

npm ERR! network request to https://registry.nlark.com/rollup-plugin-external-globals/download/rollup-plugxternal-globals-0.6.1.tgz failed, reason: getaddrinfo ENOTFOUND registry.nlark.com
npm ERR! network This is a problem related to network connectivity.
....

SOLUZIONE:
Open the file "\cloud_api_sample\source\nginx\front_page\package-lock.json", search for "registry.nlark.com" and replace them all with "registry.npmmirror.com", and execute "./update_front.sh".

```


```bash
ERRORE:

sudo docker-compose up -d
WARN[0000] /home/gianluca/Downloads/cloud_api_sample/docker-compose.yml: the attribute version is obsolete, it will be ignored, please remove it to avoid potential confusion 
[+] Running 3/6
 ✔ Network cloud_api_sample_cloud_service_bridge  Created                                                                             0.2s 
 ⠸ Container cloud_api_sample-redis-1             Starting                                                                            1.4s 
 ✔ Container cloud_api_sample-nginx-1             Started                                                                             1.4s 
 ⠸ Container cloud_api_sample-emqx-1              Starting                                                                            1.4s 
 ⠸ Container cloud_api_sample-mysql-1             Starting                                                                            1.4s 
 ✔ Container cloud_api_sample-cloud_api_sample-1  Created                                                                             0.0s 
Error response from daemon: driver failed programming external connectivity on endpoint cloud_api_sample-redis-1 (ad4144a3f29b81c8f7cd3de75bfcfded3c0fab809c29283d0caee346372cf02a): failed to bind port 0.0.0.0:6379/tcp: Error starting userland proxy: listen tcp4 0.0.0.0:6379: bind: address already in use

SOLUZIONE:
Controlla se Redis è già in esecuzione: Puoi verificare se c'è un'istanza di Redis in esecuzione sulla porta 6379:

sudo lsof -i :6379

Se esiste gà fermarla:
sudo systemctl stop redis

può avvenire la stessa coa per 1883 e 3306, in caso fare sudo kill <PID>
```

```bash
ERRORE:

mysql -u username -p
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (2)

docker-compose down
docker-compose up -d

docker exec -it cloud_api_sample-mysql-1 mysql -u root -p
#insewrire root come password

CREATE USER IF NOT EXISTS 'root'@'localhost' IDENTIFIED BY 'root';
GRANT ALL PRIVILEGES ON *.* TO 'root'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;

docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' cloud_api_sample-cloud_api_sample-1
```

## RISORSE UTILI:
- Una guida completa dornita da DJI, purtroppo in cinese: [Guida completa](https://sdk-forum.dji.net/hc/zh-cn/articles/30658553379865-docker部署常见问题及解决方案)
- Domanda nel forum di unn utente per il problema con MySQL (il secondo problema riportato nella lista sopra): [MySQL problem](https://sdk--forum-dji-net.translate.goog/hc/zh-cn/community/posts/30073925910809-docker%E9%83%A8%E7%BD%B2-api%E9%A1%B9%E7%9B%AE%E6%8A%A5%E9%94%99?input_string=MySQL&_x_tr_sl=it&_x_tr_tl=en&_x_tr_hl=it&_x_tr_pto=wapp)
- Soluzione per il problema con il servizio Web: ["network is abnormal"](https://sdk-forum.dji.net/hc/zh-cn/community/posts/34678242247705-The-network-is-abnormal-please-check-the-backend-service-and-Docker-stop-mysql?input_string=MySQL)
- Soluzione per il problema con il servizo Web: ["error 405"](https://sdk-forum.dji.net/hc/zh-cn/community/posts/25495887963673-docker部署-上云api-web端登录报405)
- 


## FILE ESEMPIO PER OTTENERE LA TELEMETRIA (con MQTT)

```python
import logging
import random
import time
from paho.mqtt import client as mqtt_client

#gatewaysn: 4LFCK5C0025SQQ
#devicesn: 1581F4BND226800BEXD9

# Definizione del broker e del topic
BROKER = '192.168.0.0'
PORT = 1883
TOPIC = "thing/product/1581F4BND226800BEXD9/osd"
CLIENT_ID = 'python-mqtt-0'  # Client ID casuale per evitare conflitti
USERNAME = 'admin'
PASSWORD = 'public'

FIRST_RECONNECT_DELAY = 1
RECONNECT_RATE = 2
MAX_RECONNECT_COUNT = 12
MAX_RECONNECT_DELAY = 60
FLAG_EXIT = False

# Funzione chiamata quando si stabilisce la connessione con il broker
def on_connect(client, userdata, flags, rc):
    if rc == 0:
        print("Connesso al broker MQTT!")
        client.subscribe(TOPIC)  # Iscrizione al topic
        print(f"Iscritto al topic: {TOPIC}")
    else:
        print(f"Connessione fallita, codice di ritorno: {rc}")

# Funzione chiamata quando il client si disconnette
def on_disconnect(client, userdata, rc):
    logging.info("Disconnesso dal broker, codice di ritorno: %s", rc)
    reconnect_count, reconnect_delay = 0, FIRST_RECONNECT_DELAY
    while reconnect_count < MAX_RECONNECT_COUNT:
        logging.info("Riconnessione in %d secondi...", reconnect_delay)
        time.sleep(reconnect_delay)
        try:
            client.reconnect()
            logging.info("Riconnesso con successo!")
            return
        except Exception as err:
            logging.error("%s. Riconnessione fallita. Ritento...", err)
        reconnect_delay *= RECONNECT_RATE
        reconnect_delay = min(reconnect_delay, MAX_RECONNECT_DELAY)
        reconnect_count += 1
    logging.info("Riconnessione fallita dopo %s tentativi. Uscita...", reconnect_count)
    global FLAG_EXIT
    FLAG_EXIT = True

# Funzione chiamata quando si riceve un messaggio
def on_message(client, userdata, msg):
    print(f'Received `{msg.payload.decode()}` from `{msg.topic}` topic')

def connect_mqtt():
    # client = mqtt_client.Client(CLIENT_ID)
    client = mqtt_client.Client(mqtt_client.CallbackAPIVersion.VERSION1, CLIENT_ID)
    client.username_pw_set(USERNAME, PASSWORD)
    client.on_connect = on_connect
    client.on_message = on_message
    client.connect(BROKER, PORT, keepalive=120)
    client.on_disconnect = on_disconnect
    return client

# Funzione principale
def run():
    logging.basicConfig(format='%(asctime)s - %(levelname)s: %(message)s',
                        level=logging.DEBUG)
    client = connect_mqtt()
    client.loop_forever()

if __name__ == '__main__':
    run()

```

```python
import logging
import random
import time
from paho.mqtt import client as mqtt_client
import json

#bid: 00000000-0000-0000-0000-000000000000
#tid: 00000000-0000-0000-0000-000000000000

#gatewaysn: 4LFCK5C0025SQQ
#devicesn: 1581F4BND226800BEXD9

# Definizione del broker e del topic
BROKER = '0.0.0.0'
PORT = 1883
TOPIC = "thing/product/1581F4BND226800BEXD9/events"
CLIENT_ID = 'python-mqtt-1'  # Client ID casuale per evitare conflitti
USERNAME = 'admin'
PASSWORD = 'public'

FIRST_RECONNECT_DELAY = 1
RECONNECT_RATE = 2
MAX_RECONNECT_COUNT = 12
MAX_RECONNECT_DELAY = 60
FLAG_EXIT = False

def on_connect(client, userdata, flags, rc):
    if rc == 0 and client.is_connected():
        print("Connected to MQTT Broker!")
    else:
        print(f'Failed to connect, return code {rc}')

def on_disconnect(client, userdata, rc):
    logging.info("Disconnected with result code: %s", rc)
    reconnect_count, reconnect_delay = 0, FIRST_RECONNECT_DELAY
    while reconnect_count < MAX_RECONNECT_COUNT:
        logging.info("Reconnecting in %d seconds...", reconnect_delay)
        time.sleep(reconnect_delay)

        try:
            client.reconnect()
            logging.info("Reconnected successfully!")
            return
        except Exception as err:
            logging.error("%s. Reconnect failed. Retrying...", err)

        reconnect_delay *= RECONNECT_RATE
        reconnect_delay = min(reconnect_delay, MAX_RECONNECT_DELAY)
        reconnect_count += 1
    logging.info("Reconnect failed after %s attempts. Exiting...", reconnect_count)
    global FLAG_EXIT
    FLAG_EXIT = True

def connect_mqtt():
    client = mqtt_client.Client(mqtt_client.CallbackAPIVersion.VERSION1, CLIENT_ID)
    client.username_pw_set(USERNAME, PASSWORD)
    client.on_connect = on_connect
    client.connect(BROKER, PORT, keepalive=120)
    client.on_disconnect = on_disconnect
    return client

def publish(client):
    msg_count = 0
    while not FLAG_EXIT:
        msg_dict = {
            "data": {
                "output": {
                    "ext": {
                        "camera_mode": 3
                    },
                    "progress": {
                        "current_step": 0,
                        "percent": 100
                    },
                    "status": "ok"
                },
                "result": 0
            }
        }
        msg = json.dumps(msg_dict)
        if not client.is_connected():
            logging.error("publish: MQTT client is not connected!")
            time.sleep(1)
            continue
        result = client.publish(TOPIC, msg)
        # result: [0, 1]
        status = result[0]
        if status == 0:
            print(f'Send `{msg}` to topic `{TOPIC}`')
        else:
            print(f'Failed to send message to topic {TOPIC}')
        msg_count += 1
        time.sleep(1)

def run():
    logging.basicConfig(format='%(asctime)s - %(levelname)s: %(message)s',
                        level=logging.DEBUG)
    client = connect_mqtt()
    client.loop_start()
    time.sleep(1)
    if client.is_connected():
        print("IF")
        publish(client)
    else:
        client.loop_stop()

if __name__ == '__main__':
    run()
```







# MOBILE SDK

A grandi linee viene seguito questo tutorial:
[https://developer.dji.com/doc/mobile-sdk-tutorial/en/quick-start/tion download markdown](https://developer.dji.com/doc/mobile-sdk-tutorial/en/quick-start/run-sample.html)

notion notionffffdffdff[un-sample.html](https://developer.dji.com/doc/mobile-sdk-tutorial/en/quick-start/run-sample.html)

con l’implementazione di alcune accortezze tratte da:

https://sdk-forum.dji.net/hc/en-us/articles/5983558993433-Chapter-2-The-Sample

- Installare Android Studio Giraffe | 2022.3.1 Patch 4: https://developer.android.com/studio/archive?hl=it
- scaricare https://github.com/dji-sdk/Mobile-SDK-Android-V5/tree/dev-sdk-main
- Importare in un nuovo progetto solo la cartella  android-sdk-v5-as ma tenere tutto (sample e uxdx servono come moduli, non eliminarli)
- Usare Java 17 o superiore (File/Project Structure/SDK Location e cliccare su Gradle Settings alla fine della pagina → quindi selezionare Gradle JDK)
- Creare una nuova applicazione su dji developer [https://developer.dji.com/](https://developer.dji.com/user/apps/#all) usare come package name “com.dji.sampleV5.aricraft”

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