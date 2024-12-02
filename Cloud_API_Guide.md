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
1. Il primo file implementa un client MQTT utilizzando la libreria paho-mqtt, si connette a un broker MQTT definito da un IP e una porta, si iscrive a un topic specifico e gestisce la connessione e la disconnessione del client, con tentativi automatici di riconnessione in caso di disconnessione. Quando riceve un messaggio dal broker, lo stampa sul terminale, può essere usato per ottenere i dati della telemetria in tempo reale.
    ```python
    import logging
    import time
    from paho.mqtt import client as mqtt_client
    
    # Definizione del broker e del topic
    BROKER = '192.168.0.0'
    PORT = 1883
    TOPIC = "thing/product/***********/osd" #inserire il DeviceSN corretto (lo si trova anche nel telecomando DJI)
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

