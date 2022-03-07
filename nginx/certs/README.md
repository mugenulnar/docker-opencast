# LINUX Version

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


# WINDOWS Version
- [Enlace Guia Oficial](https://wiki.openssl.org/index.php/Binaries)

## Install OpenSSL binaries
1. Enter on the first enrty [SLProject](https://slproweb.com/products/Win32OpenSSL.html)
2. Scroll down and select one of the *NON-LIGHT Edition* 
3. Click on the EXE or MSI option
4. Install it.
5. On the installation select "Use windows directory" (All defaults installation settings)
6. (Optional) Done (or not) to OpenSSL project.

## Execute the binaries
1. Run CMD as administrator
2. Go to "C:\Program Files\OpenSSL-Win64" -> `cd " C:\Program Files\OpenSSL-Win64\bin"`
3. Follow the same steps as in the Linux version