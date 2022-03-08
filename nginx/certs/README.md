# LINUX Version

## Generate Private Key 
openssl genrsa -des3 -out <key_file_name>.key 2048

## Create a Certificate Request (CSR)
openssl req -key <key_file_name>.key -new -out <csr_file_name>.csr

## Create the Self Signed Certificate
openssl x509 -signkey <key_file_name>.key -in <csr_file_name>.csr -req -days 365 -out <crt_file_name>.crt

## Create a Self Root CA
openssl req -x509 -sha256 -days 1825 -newkey rsa:2048 -keyout rootCA.key -out rootCA.crt

## Create a configuration file for the CSR CA
```bash
#<ext_file_name>.ext
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
subjectAltName = @alt_names
[alt_names]
DNS.1 = <local_domain>.local
```

## Sign the CA-Certificate
openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in <csr_file_name>.csr -out <crt_file_name>.crt -days 365 -CAcreateserial -extfile <ext_file_name>.ext

## (Optional) Convert Certificate to PFX Certificate Format
openssl pkcs12 -inkey <key_file_name>.key -in <crt_file_name>.crt -export -out <pfx_file_name>.pfx


# WINDOWS Version
- [Official Guide](https://wiki.openssl.org/index.php/Binaries)

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
4. Remove the phrase on the KEY certificate-> `openssl rsa -in <key_file>.key -out <NEW_key_file>.key`
