
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

```
#!/bin/bash



openssl genpkey -algorithm RSA -out server.key

# Generate a certificate signing request (CSR)

openssl req -new -newkey rsa:2048 -nodes -sha256 -keyout server.key -out server.csr -config req.conf



# Generate a self-signed certificate valid for 365 days

openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt -extfile req.conf -extensions v3_req



# Display the generated certificate information

openssl x509 -in server.crt -noout -text



openssl x509 -in server.crt -out server.pem -outform PEM



# Clean up the certificate signing request

rm server.csr

rm server.crt



echo "SSL certificate and private key generated successfully!"

echo "SSL certificate and private key generated successfully!"



```
