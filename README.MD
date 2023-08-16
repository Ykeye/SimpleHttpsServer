
## Here I present an example of a simple HTTPS server in python that can be used for various testing purposes.

```
from http.server import HTTPServer, SimpleHTTPRequestHandler
import ssl
import threading


def run_https_server():
        https_server = HTTPServer(('', 443), SimpleHTTPRequestHandler)
        https_server.socket = ssl.wrap_socket(https_server.socket,keyfile="./server.key",certfile="./server.pem",server_side=True)
        https_server.serve_forever()


def run_http_server():
        http_server = HTTPServer(('', 80), SimpleHTTPRequestHandler)
        http_server.serve_forever()


# Start the HTTPS server in a separate thread
https_thread = threading.Thread(target=run_https_server)
https_thread.start()

# Start the HTTP server in the main thread
run_http_server()

```


You can generate teh certificate and the key using this bash script also
But you have to create the ```req.conf``` config file first


```
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no
[req_distinguished_name]
C = RU
ST = MOW
L = Moscow
O = Akon
OU = Support
CN = www.test.ru
[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = www.test.ru
DNS.2 = test.ru
DNS.3 = www.test.net
DNS.4 = test.net
	
```



```
#!/bin/bash



openssl genpkey -algorithm RSA -out server.key

openssl req -new -newkey rsa:2048 -days 365 -extensions v3_ca -subj "/C=RU/ST=mow/L=Moscow/O=Akon/OU=Cpsupport/CN=Authority/" -nodes -x509 -sha256 -set_serial 0 -keyout root_key.key -out root_cer.cer


# Generate a certificate signing request (CSR)

openssl req -new -newkey rsa:2048 -nodes -sha256 -keyout server.key -out server.csr -config req.conf



# Generate a self-signed certificate valid for 365 days

openssl x509 -req -sha256 -CAcreateserial -days 365 -in server.csr -CA root_cer.cer -CAkey root_key.key -out server.cer -extfile req.conf -extensions v3_req 



# Display the generated certificate information

openssl x509 -in root_cer.cer -noout -text
openssl x509 -in server.cer -noout -text

openssl x509 -in server.cer -out server.pem -outform PEM
openssl x509 -in root_cer.cer -out root_cer.pem -outform PEM

echo -e "\n\n\n"

openssl verify -CAfile root_cer.pem server.pem

cat server.pem > bundle.pem
cat root_cer.pem >> bundle.pem

echo Finised a Bundle

openssl verify -CAfile root_cer.cer bundle.pem



# Clean up the certificate signing request

rm server.csr
rm server.cer
rm root_cer.cer


echo "SSL certificate and private key generated successfully!"
echo "Import the bundle.pem file by itself and split it after using text editor or just copy server.pem and root_cer.pem to your local machine and import it into Trusted Ca Registry"
```
