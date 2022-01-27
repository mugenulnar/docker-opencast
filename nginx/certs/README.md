# How to generate a Self Certificate[Link here](https://www.baeldung.com/openssl-self-signed-cert)

## Generate Private Key 
openssl genrsa -des3 -out opencast.key 2048

## Create a Certificate Request (CSR)
openssl req -key opencast.key -new -out opencast.csr

## Create the Self Signed Certificate
openssl x509 -signkey opencast.key -in opencast.csr -req -days 365 -out opencast.crt

## Create a Self Root CA
openssl req -x509 -sha256 -days 1825 -newkey rsa:2048 -keyout rootCA.key -out rootCA.crt

## Create a configuration file for the CSR CA
```bash
#opencast-dsa.ext
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
subjectAltName = @alt_names
[alt_names]
DNS.1 = opencast-dsa.local
```

## Sign the CA-Certificate
openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in opencast.csr -out opencast.crt -days 365 -CAcreateserial -extfile opencast-dsa.ext

## (Optional) Convert Certificate to PFX Certificate Format
openssl pkcs12 -inkey opencast.key -in opencast.crt -export -out opencast.pfx
