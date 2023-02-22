# ESP32GcloudIoTCoreTest
 Google IoT Core ESP32 TESTED CODE
 This is an updated and tested version of original code to Connect an ESP32 running Arduino Frame Work to Google IoT core to be deprecated on 2023... i know its 2022 but its never too late.... 
 by Edwin Kestler.

 IDE: Visual Studio Code
 Extension: Platform.io
 External Libraries: 
            googlecloudplatform/Google Cloud IoT Core JWT@^1.1.11 (google IoT Core)
	        256dpi/MQTT@^2.5.0 (Arduino MQTT)
 Configuration Library:
            ciotc_config.h
 
 Other Tools needed: 
        Google Cloud CLI (SDK):
                                Easy intall by Winget: (open cmd): in windows command promt "C:> winget install -e --id Google.CloudSDK" (hit Enter).
        OpenSSL: 
                                Easy intall by Winget: (open cmd):  in windows command promt "C:> winget install -e --id ShiningLight.OpenSSL" (hit enter).
        
1) configure your Google IoT Core Registry, location(zone), keys, and Pub/sub Topics.
open your Google Cloud SDK terminal:
 //intiate Google cloud. 
 :> gcloud init 
 (configure your identity, project, zone (example: us-central1-a))

 //create the ca_certificate files
 :>openssl genpkey -algorithm RSA -out ca_private_registry.pem -pkeyopt rsa_keygen_bits:2048
 :>openssl req -x509 -new -nodes -key ca_private_registry.pem -sha256 -out ca_cert_registry.pem -subj "/CN=unused"
 
 //register the credentials you entered. 
 :>gcloud iot registries credentials create --path=ca_cert_registry.pem  --project=<YOUR_PROJECT_ID>  --registry=<NAME_OF_YOUR_REGISTRY> --region=<YOUR_REGION> (ex: us-central1) <-- NOT us-cenral1-a.

 //NOTE: if error similar to openssl not a valid command: chek "/your-install-folder/openssl/bin" is configured in windos ENVIROMENT VARIABLES "PATH" THEN:
 
 //create a Pub/sub Publish topic
 :> gcloud pubsub topics create <YOUR_PUB/SUB_PUBLISH_TOPIC> --project=<YOUR_PROJECT_ID>

 //create a Pub/sub Subscribe Topic under your Publish Topic
 :>gcloud pubsub subscriptions create <YOUR_PUB/SUB_SUBSCRIBE_TOPIC> --topic=<YOUR_PUB/SUB_PUBLISH_TOPIC>

 //if you have NOT configured a recistry on the google cloud web console:
 :>gcloud iot registries create <NAME_OF_YOUR_REGISTRY> --region=<YOUR_REGION> --event-notification-config=topic=<YOUR_PUB/SUB_PUBLISH_TOPIC>

 //if you have configured a recistry on the google cloud web console:
  :> gcloud iot registries describe <NAME_OF_YOUR_REGISTRY> --project=<YOUR_PROJECT_ID>  --region=<YOUR_REGION> (ex: us-central1) <-- NOT us-cenral1-a.

//Generate an Eliptic Curve (EC) private / public key pair:
    :> openssl ecparam -genkey -name prime256v1 -noout -out ec_private_device1.pem
    :> openssl req -new -sha256 -key ec_private_device1.pem -out ec_cert_device1.csr -subj "/CN=unused-device"
    :> openssl x509 -req -in ec_cert_device1.csr -CA ca_cert_registry.pem -CAkey ca_private_registry.pem -CAcreateserial -sha256 -out ec_cert_device1.pem

//create a device.
  :> gcloud iot devices create <NAME_OF_YOUR_DEVICE> --project=<YOUR_PROJECT_ID> --region=<YOUR_REGION> --registry=<NAME_OF_YOUR_REGISTRY> --public-key path=/path/to/ec_cert_device1.pem,type=es256-x509-pem

//In Web console upload Certificate to device in google cloud iot registry, in authentication tab. choose add public key and choose file:
 "ec_cert_device1.pem" as type ES256_X509

//Print the Private part of the certificate generated and copy to the variable "private_key" in "ciotc_config.h" file 
should be:
        C:\>openssl ec -in ec_private_device1.pem -noout -text
        //output:
        read EC key
        Private-Key: (256 bit)
        priv:                                               //copy the lines under this (ctrl+c)
            00:6c:f7:14:6b:0f:1f:f1:8a:74:05:0c:a3:37:5b:   //this
            5e:64:15:57:6c:90:c1:b9:bd:58:02:49:de:b1:dd:   //this
            4f:23                                           //thi
        pub:
            04:22:a3:85:44:e0:2d:2d:fc:b5:94:52:65:95:87:
            d5:7f:00:47:8d:08:d2:9e:75:66:5a:b4:8c:8f:fe:
            ad:be:64:65:2d:5b:e1:9e:91:59:72:f8:cb:06:e3:
            9a:32:1d:be:eb:55:09:00:4a:80:00:ac:72:00:61:
            3f:90:e6:55:85
        ASN1 OID: prime256v1
        NIST CURVE: P-256
Copy to file rememnber variable expets const char so qoutes should be included after each line copied like so:
            "00:6c:f7:14:6b:0f:1f:f1:8a:74:05:0c:a3:37:5b:"
            "5e:64:15:57:6c:90:c1:b9:bd:58:02:49:de:b1:dd:"
            "4f:23"

//Download TLS certificates for permanent mqtt google api endpoint: mqtt.2030.ltsapis.goog
Primary certificate: at https://pki.goog/gtsltsr/gtsltsr.crt 
Secondary certificate: at https://pki.goog/gsr4/GSR4.crt

//transform .crt files into pem files by typing in console:
:>openssl x509 -inform DER -in gtsltsr.crt -out primary.pem -text
:>openssl x509 -inform DER -in GSR4.crt -out secondary.pem -text

//open the files in visual studio code and copy each certificate beggining from -----BEGIN CERTIFICATE----- all the way to -----END CERTIFICATE----- selecting it and then Ctrl+c and paste each corresponding certificate on its certificate variable "primary_ca" and "backup_ca" in the "ciotc_config.h" file under the "include" folder in Visual stuio code explorer. 

//remenber this variables are const char* so remember to add Qoutes and an "end of line" /n on each line of the certificate like So:
            "-----BEGIN CERTIFICATE-----\n"

//This demo trnasmits the RSSI (wifi signal strenght as the "data" for this sensor) you can look at the data been transmitted by pulling the pub/sub messages from the subscribe topic in the SDK cli by typing the command:
:>gcloud pubsub subscriptions pull --limit 500 --auto-ack <your-subscription-name>

this code is based on other great turarials and libraries like:
thxssio, https://github.com/Thxssio/codes/tree/e3fa62f42e5f1a481af33e96c6a7cb42b28c5b21/libraries/Google_Cloud_IoT_Core_JWT
leozz37, iot-monitoring-gcp-grafana: https://github.com/leozz37/iot-monitoring-gcp-grafana
Ludovic Emo, How to send ESP32 telemetry to Google Cloud IoT Core https://ludovic-emo.medium.com/how-to-send-esp32-telemetry-to-google-cloud-iot-core-caf1a952020d
Gus Class, Accessing Cloud IoT Core from Arduino: https://medium.com/@gguuss/accessing-cloud-iot-core-from-arduino-838c2138cf2b

Google documentation on IoT core canbe found at:
Ejemplos de la CLI de gcloud, https://cloud.google.com/iot/docs/gcloud-examples
Publicar en el puente MQTT,  https://cloud.google.com/iot/docs/how-tos/mqtt-bridge#iot-core-mqtt-publish-cpp
Publicar en el puente MQTT usando un TLS de termino largo, https://cloud.google.com/iot/docs/how-tos/mqtt-bridge#using_a_long-term_mqtt_domain

For Trouble shooting threads on libraries and certificates:
Google CA Certificate verification failing #4909, https://github.com/espressif/arduino-esp32/issues/4909
ESP32-lwmqtt example does not works with latest WiFiClientSecure library #221, https://github.com/GoogleCloudPlatform/google-cloud-iot-arduino/issues/221














