#To enable basic auth for node_exporter create a hashed password first.
Example:

apt install apache2-utils
htpasswd -nBC 12 ""
$2$xxxxxx

Then update /etc/node_exporter/config.yml and restart node_exporter.
>
basic_auth_users:
  prometheus: $2$xxxxxx
  #user: #hashpass

Also, update prometheus.yml

  - job_name: "node_exporter"
    basic_auth:
      username: "prometheus"
      password: "P@ssw0rd" 
    static_configs:
      - targets: ["localhost:9100"]
        labels:
          app: "node_scrape"

//////////////////////////////////////////////////////////////////////////////////

TLS Config for Node exporter.
Create TLS cert using openssl or any other.

Example: 

openssl req -new -newkey rsa:2048 -days 3650 -nodes -x509 -keyout node_exporter.key -out node_exporter.crt -addext "subjectAltName = DNS:localhost"

change ownsership if needed for both crt and key,

add this to node_exporter.yml

"
tls_server_config:
  cert_file: /etc/node_exporter/node_exporter.crt 
  key_file: /etc/node_exporter/node_exporter.key
"

Restart node_exporter.

Then copy the cert to /etc/prometheus (either same machine or if another), and change ownership to promethues
add this data to prometheus.yml job 
"
    scheme: "https"
    tls_config:
      ca_file: /etc/prometheus/node_exporter.crt
      insecure_skip_verify: true
"