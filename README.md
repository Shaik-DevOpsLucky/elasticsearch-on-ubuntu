
# 📌 Secure Elasticsearch 8.x Installation — Clean & Hardened

---

## ✅ 1️⃣ Install Dependencies
```bash
sudo apt update
sudo apt install apt-transport-https ca-certificates wget curl gnupg -y
```

---

## ✅ 2️⃣ Import GPG Key & Add Elastic APT Repo
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
```

---

## ✅ 📌 Important (Auto-generated Password Notice)
When executing the above step, **Elasticsearch’s security auto-configuration generates an admin password automatically** for the built-in `elastic` superuser.

Example output:
```
---------------- Security auto-configuration information ----------------

Authentication and authorization are enabled.
TLS for the transport and HTTP layers is enabled and configured.

The generated password for the elastic built-in superuser is : FO-LToW4rHeS57_3Q924

If this node should join an existing cluster, you can reconfigure this with
'/usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <token-here>'

You can complete the following actions at any time:

Reset the password of the elastic built-in superuser with
'/usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic'

Generate an enrollment token for Kibana instances with
'/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana'

Generate an enrollment token for Elasticsearch nodes with
'/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node'

---------------------------------------------------------------------------
```

✅ **Note down this password carefully** — you’ll need it later.

---

## ✅ 3️⃣ Install Elasticsearch
```bash
sudo apt update
sudo apt install elasticsearch -y
```

---

## ✅ 4️⃣ Secure Configuration Hardening
Edit `/etc/elasticsearch/elasticsearch.yml`:
```bash
sudo vi /etc/elasticsearch/elasticsearch.yml
```
Apply these important settings:
```yaml
network.host: 127.0.0.1
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.http.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.http.ssl.keystore.path: certs/http.p12
xpack.security.http.ssl.truststore.path: certs/http.p12
xpack.ml.enabled: false
xpack.monitoring.collection.enabled: true
xpack.watcher.enabled: false
```

Check existing cert directory:
```bash
ls -l /etc/elasticsearch/certs/
```

---

## ✅ 5️⃣ Generate Security Certificates (if not already done)
- Reset elastic password (if needed)
```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

- Generate HTTP and CA certificates:
```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil http
sudo /usr/share/elasticsearch/bin/elasticsearch-certutil ca
```

- Move and secure certificates:
```bash
sudo mkdir -p /etc/elasticsearch/certs
sudo mv *.p12 /etc/elasticsearch/certs/
sudo chown -R elasticsearch:elasticsearch /etc/elasticsearch/certs
sudo chmod 600 /etc/elasticsearch/certs/*.p12
```

---

## ✅ 6️⃣ Enable and Start Elasticsearch
```bash
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```

---

## ✅ 7️⃣ Harden System User & Directory Permissions
```bash
sudo usermod --lock root
sudo usermod --shell /usr/sbin/nologin elasticsearch
sudo chmod 750 /etc/elasticsearch
sudo chmod 640 /etc/elasticsearch/elasticsearch.yml
```

---

## ✅ 8️⃣ Disable Unused Plugins
- List installed plugins:
```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-plugin list
```

- Remove unnecessary plugins:
```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-plugin remove <plugin-name>
```

---

## 📌 🔒 Recap: Secure Installation Summary
✅ TLS encryption for HTTP & transport  
✅ User password authentication enabled  
✅ Restricted to localhost or limited IPs  
✅ Directory & file permissions hardened  
✅ Disabled unused plugins  
✅ Locked system users  

---

## ✅ Final Test
- Local secure connection:
```bash
curl -k -u elastic https://localhost:9200
```

- Remote (with SSL cert verification):
```bash
curl --cacert /path/to/http_ca.crt -u elastic https://your-server-ip:9200
```

---

## ✅ Optional Useful Commands
- Start service manually:
```bash
sudo systemctl start elasticsearch.service
```
- Enable on system boot:
```bash
sudo systemctl enable elasticsearch.service
```

# Prepared by:
*Shaik Moulali*
