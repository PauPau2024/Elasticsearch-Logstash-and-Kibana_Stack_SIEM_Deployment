Setting up the **ELK Stack** (Elasticsearch, Logstash, and Kibana) allows you to collect, store, search, and visualize logs in real-time. Below is a step-by-step guide to set it up on a Linux-based system (like Ubuntu 20.04).

---

## 🔧 Prerequisites:

* OS: Ubuntu 20.04 or similar
* Java 11+ (required for Logstash)
* 4 GB RAM minimum
* `sudo` access

---

## 1. **Install Elasticsearch**

### Add the Elastic APT repo:

```bash
sudo apt update
sudo apt install apt-transport-https ca-certificates gnupg -y
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update
```

### Install Elasticsearch:

```bash
sudo apt install elasticsearch -y
```

### Enable & start the service:

```bash
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```

---

## 2. **Install Logstash**

```bash
sudo apt install logstash -y
```

You’ll configure this later using `.conf` files.

---

## 3. **Install Kibana**

```bash
sudo apt install kibana -y
```

Enable and start:

```bash
sudo systemctl enable kibana
sudo systemctl start kibana
```

Open Kibana at `http://localhost:5601` in your browser.

---

## 4. **Basic Configuration**

* **Elasticsearch**: `/etc/elasticsearch/elasticsearch.yml`

  ```yaml
  network.host: localhost
  ```
* **Kibana**: `/etc/kibana/kibana.yml`

  ```yaml
  server.host: "localhost"
  elasticsearch.hosts: ["http://localhost:9200"]
  ```

---

Here's a **complete step-by-step guide to set up the Kibana keystore**, which is used to securely store secrets like passwords, encryption keys, tokens, etc., **without putting them in `kibana.yml`**.

---

## ✅ 1. **Navigate to the Kibana `bin` directory**

```bash
cd /usr/share/kibana/bin
```

---

## ✅ 2. **Create the Kibana keystore**

If you haven’t already:

```bash
./kibana-keystore create
```

Expected output:

```
Created Kibana keystore in /usr/share/kibana/data/kibana.keystore
```

---

## ✅ 3. **Add secure settings**

You can now securely add secrets. Here are the most commonly used ones:

### 🔐 Add `xpack.security.encryptionKey`

```bash
echo "your_32_char_encryption_key" | ./kibana-keystore add xpack.security.encryptionKey --stdin
```

Example:

```bash
echo "57694b5b18cc737d66191bd562d14776" | ./kibana-keystore add xpack.security.encryptionKey --stdin
```

---

### 🔐 (Optional) Add other secure values:

#### Encrypted Saved Objects key:

```bash
echo "another_32_char_key" | ./kibana-keystore add xpack.encryptedSavedObjects.encryptionKey --stdin
```

#### Reporting key:

```bash
echo "reporting_32_char_key" | ./kibana-keystore add xpack.reporting.encryptionKey --stdin
```

> All encryption keys must be exactly **32 characters** long.

---

## ✅ 4. **Verify keys stored**

To see what keys are stored (but not their values):

```bash
./kibana-keystore list
```

---

## ✅ 5. **Restart Kibana**

```bash
sudo systemctl restart kibana
```

---

## 🔒 Notes:

* The keystore is stored at: `/usr/share/kibana/data/kibana.keystore`
* Do **not** manually edit this file.
* You can use the keystore to avoid exposing secrets in plain text in `kibana.yml`.

---

Would you like to automate this process via a script or add keys programmatically inside a Docker container or deployment pipeline?


---


Absolutely! Here's a simple and clear **tutorial for setting up and configuring UFW (Uncomplicated Firewall)** to allow access to Elasticsearch and Kibana while ensuring SSH access is maintained.

---

# 🔥 **UFW Firewall Setup for ELK Stack (Ubuntu)**

This guide explains how to:

* Enable UFW
* Allow access to Elasticsearch (9200)
* Allow access to Kibana (5601)
* Ensure you don’t lose SSH access

---

## 📋 **Step-by-Step Instructions**

### 🧱 Step 1: Check UFW Status

```bash
sudo ufw status
```

If output is:

```
Status: inactive
```

→ The firewall is installed but not yet active.

---

### 🔐 Step 2: Allow SSH Access (Prevent Lockout)

This step is **critical** if you're using a remote server via SSH.

```bash
sudo ufw allow OpenSSH
```

Or manually:

```bash
sudo ufw allow 22/tcp
```

---

### 🔥 Step 3: Enable the Firewall

```bash
sudo ufw enable
```

You’ll see:

```
Command may disrupt existing ssh connections. Proceed with operation (y|n)?
```

Type `y` and press Enter.

> ✅ UFW is now active and will persist on startup.

---

### 🚪 Step 4: Allow ELK Stack Ports

#### ➤ Allow Elasticsearch (port 9200)

```bash
sudo ufw allow 9200/tcp
```

#### ➤ Allow Kibana (port 5601)

```bash
sudo ufw allow 5601/tcp
```

This allows external access to the respective services.

---

### 🧪 Step 5: Verify Firewall Rules

```bash
sudo ufw status verbose
```

Example output:

```
Status: active

To                         Action      From
--                         ------      ----
22/tcp (OpenSSH)           ALLOW       Anywhere
9200/tcp                   ALLOW       Anywhere
5601/tcp                   ALLOW       Anywhere
```

---

## ✅ Done!

You’ve successfully:

* Enabled UFW
* Preserved SSH access
* Allowed necessary ports for Elasticsearch and Kibana

---

## 🔒 Optional: Restrict Access to Specific IP

Instead of exposing Kibana/Elasticsearch to the whole internet, allow only your IP:

```bash
sudo ufw allow from <your-ip-address> to any port 5601 proto tcp
sudo ufw allow from <your-ip-address> to any port 9200 proto tcp
```

Then delete the wide-open rules:

```bash
sudo ufw delete allow 5601/tcp
sudo ufw delete allow 9200/tcp
```

---

Let me know if you'd like this in **Markdown**, **PDF**, or for a **Docker-based ELK setup**.


## 5. **Send Logs with Logstash**

Create a basic Logstash pipeline config file:

```bash
sudo nano /etc/logstash/conf.d/simple.conf
```

Example:

```conf
input {
  file {
    path => "/var/log/syslog"
    start_position => "beginning"
  }
}

filter {
  grok {
    match => { "message" => "%{SYSLOGTIMESTAMP:timestamp} %{SYSLOGHOST:host} %{DATA:program}: %{GREEDYDATA:log_message}" }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "syslog-%{+YYYY.MM.dd}"
  }
}
```

Start Logstash:

```bash
sudo systemctl restart logstash
```

---

## 6. **Verify the Setup**

* Test Elasticsearch: `curl http://localhost:9200`
* Access Kibana: [http://localhost:5601](http://localhost:5601)
* Go to **Discover** and search logs

---

## 7. **Optional: Use Filebeat for Lightweight Log Shipping**

```bash
sudo apt install filebeat -y
sudo filebeat modules enable system
sudo filebeat setup
sudo systemctl start filebeat
```

---

## 🧠 Tips:

* Keep your ELK stack version consistent (e.g., 8.x).
* Secure the stack using basic authentication and SSL.
* Use dashboards in Kibana to visualize data effectively.
* Use Filebeat for production instead of Logstash for direct log shipping.

---

Let me know if you want a **Docker Compose setup**, **cloud deployment**, or **secured ELK setup** with SSL and authentication.
