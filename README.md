# WiFi-Authentication-with-EAP-SIM
Introduction

EAP Subscriber Identity Module (EAP-SIM) use the subscriber identity module (sim) from Global System for Mobile Communications (GSM) for authentication and session key distribution. Also EAP-SIM can be used in a WPA enterprise network (WPA-802.1X). Our goal is to obtain access to a secure WiFi through our smartphone without a password by using EAP-SIM authentication method.  Our implementation consists of a freeradius server, an access point (netfaster 3) and an android 5.1 smartphone. 

![image](https://user-images.githubusercontent.com/35136550/34718799-cc2c0592-f540-11e7-9233-917cbbcd3be4.png)

                        Implementation Scheme

Authentication Triplets

In EAP-SIM the communication between the SIM card and the Authentication Centre (AuC)replaces the need for a pre-established password between the client and the AAA server. The AuC generates the authentication triplets (RAND, Kc, SRES), the RAND variable transferred in the sim card and in combination with the secret key Ki generates the SRES and the Kc using the A3 and A8 algorithms respectively. In order to authenticate with EAP-SIM we have to extract the authentication triplets from the sim card as we aren't a mobile network operator to have an AuC which knows the secret key Ki and generates the authentication triplets. The triplet consists of a key Kc, a random variable RAND and a variable SRES. We extracted the triplets with Osmocom Simtrace hardware board by reading the APDU communication but it's easier to use a smart card reader instead. The triplets should be hard-coded into freeradius server in order to authenticate the smartphone.
 
 ![image](https://user-images.githubusercontent.com/35136550/34718864-0914fa7c-f541-11e7-8f1a-70d649f8d340.png)
 
                        Osmocom Simtrace sample output
Configuration Files

users

In this file, for every IMSI we should put at least three authentication triplets. The first line in the bellow image is made up of the IMSI (12XXXXX18), the Mobile Network Code (mnc), 000 is our provider ‘s code, the Mobile Country Code (mcc), 202 is our country ‘s code and the EAP type which is used (SIM).
 
![image](https://user-images.githubusercontent.com/35136550/34718867-0f1398b6-f541-11e7-95a5-8a622d07d7e9.png)

eap

In this file we should define sim as the default EAP type.

default_eap_type = sim 

default

The main configuration file that calls the other files. 

![image](https://user-images.githubusercontent.com/35136550/34718986-72131c2a-f541-11e7-9db5-dd87c5f5ab39.png)

We forced to insert the above lines, as the variable User-Name we were receiving from the AP, hasn’t the ending “org”, which exists in the variable Identity as we can see bellow.

![image](https://user-images.githubusercontent.com/35136550/34718995-7e4fe22a-f541-11e7-8fc4-64bcacb19ff6.png)

And the freeradius server were terminating with the bellow error.

![image](https://user-images.githubusercontent.com/35136550/34718998-82120e4c-f541-11e7-8dd2-aedc77114dd6.png)

The next change we made in this file was to put under the files command the above code that will read the authentication triplets which correspond in the user ‘s IMSI who tries to connect.
 
![image](https://user-images.githubusercontent.com/35136550/34719003-89b352fa-f541-11e7-91fd-4d0d386abcc1.png)

clients.conf

In this file we should put the AP ‘s IP and a secret key which is shared between the radius server and the AP.

![image](https://user-images.githubusercontent.com/35136550/34719336-20a7b344-f543-11e7-89c3-cec651daa8a4.png)

After that the only thing we have to do is to configure the AP to use 802.1X and the smartphone to use SIM as the EAP method when it tries to connect in the WiFi.

Access Point Configuration

![image](https://user-images.githubusercontent.com/35136550/34719019-9f832f56-f541-11e7-9b86-8937d0a10b59.png)

![image](https://user-images.githubusercontent.com/35136550/34719022-a2bab798-f541-11e7-87e6-5ebe3ed2f712.png) 

Proof of Concept

Device that supports EAP-SIM authentication (Moto G Android 5.1 Smartphone in this case)
  
![image](https://user-images.githubusercontent.com/35136550/34719029-ab197e38-f541-11e7-9cd1-267d5b62f510.png)
![image](https://user-images.githubusercontent.com/35136550/34719031-aebe342a-f541-11e7-8d0e-7636b2860530.png)

Security Considerations

IMSI leakage

The IMSI transferred unencrypted so, someone can see the user ‘s IMSI during the authentication process by simply monitoring the traffic.

![image](https://user-images.githubusercontent.com/35136550/34719038-ba317b8c-f541-11e7-8e65-6d2f8ed017a9.png)

![image](https://user-images.githubusercontent.com/35136550/34719049-c69f97b4-f541-11e7-800b-bf7252273189.png)
