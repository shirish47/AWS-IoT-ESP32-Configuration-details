# AWS-IoT-ESP32-Configuration-details

### Generating own device certificates using own root CA certificates

##### Step 1 Create a CA certificate.
1. Generate a key pair.

```openssl genrsa -out rootCA.key 2048```

2. Use the private key from the key pair to generate a CA certificate.

```openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.pem```

##### Step 2 Register CA certificate.
1. Get a registration code from AWS IoT. This code will be used as the Common Name of the private key verification certificate.

```aws iot get-registration-code```

2. Generate a key pair for the private key verification certificate. 

```openssl genrsa -out verificationCert.key 2048``` 

3. Create a CSR for the private key verification certificate. Set the Common Name field of the certificate to your registration code.

```openssl req -new -key verificationCert.key -out verificationCert.csr ```

You will be prompted for some information, including the Common Name, for the certificate.

```
Country Name (2 letter code) [AU]:
State or Province Name (full name) []:
Locality Name (for example, city) []:
Organization Name (for example, company) []:
Organizational Unit Name (for example, section) []:
Common Name (e.g. server FQDN or YOUR name) []:XXXXXXXXXXXXMYREGISTRATIONCODEXXXXXX
Email Address []: 
```

4. Use the CSR to create a private key verification certificate

``` openssl x509 -req -in verificationCert.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out verificationCert.crt -days 500 -sha256```

5. Register the CA certificate with AWS IoT. Pass in the CA certificate and the private key verification certificate to the register-ca-certificate CLI command.

``` aws iot register-ca-certificate --ca-certificate file://rootCA.pem --verification-cert file://verificationCert.crt ```

(above command differs in some AWS documents but this if you are manually uploading file then use .crt as amazon ask for verificationCert.crt file).

6. Use the update-certificate CLI command to activate the CA certificate . 

``` aws iot update-ca-certificate --certificate-id xxxxxxxxxxx --neww-status ACTIVE ```

##### Step 3 Create Device Certificate using the CA certificate.
You can use a CA certificate registered with AWS IoT to create a device certificate. The device certificate must be registered with AWS IoT before use. 

1. Generate a key pair.

```openssl genrsa -out deviceCert.key 2048``` 

2. Create a CSR for the device certificate.

```openssl req -new -key deviceCert.key -out deviceCert.csr```

You will be prompted for some information, as shown here.
```
Country Name (2 letter code) [AU]:
State or Province Name (full name) []:
Locality Name (for example, city) []:
Organization Name (for example, company) []:
Organizational Unit Name (for example, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
```

3. Create a device certificate from the CSR. 

``` openssl x509 -req -in deviceCert.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial - out deviceCert.crt -days 500 -sha256 ```
(some docs suggest deviceCert.pem)

Note You must use the CA certificate registered with AWS IoT to create device certificates. 
If you have more than one CA certificate (with the same subject field and public key) registered in your AWS account, you must specify the CA certificate used to create the device certificate when you register your device certificate.

##### Step 4 Register Device Certificate.

1. Register a device certificate.

``` aws iot register-certificate --certificate-pem file://deviceCert.pem --ca-certificatepem file://rootCA.pem ```

2. Use the update-certificate CLI command to activate the device certificate .

```aws iot update-certificate --certificate-id xxxxxxxxxxx --new-status```


For using with ESP32 Examples from IDF rename
##### rootCA.pem to aws-root-ca.pem, deviceCert.crt to certificate.pem.crt and deviceCert.key to private.pem.key

##### Step 5 attach policies things to Device Certificate.
Do not forget to attach policies and Thing to certificate. This can be done manually over AWS IoT Console.
 

