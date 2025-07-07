Setting up the **ELK Stack** (Elasticsearch, Logstash, and Kibana) allows you to collect, store, search, and visualize logs in real-time. Below is a step-by-step guide to set it up on a Linux-based system (like Ubuntu 20.04). You can adapt it for Docker or cloud as needed.

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

## 4. **Basic Configuration (Optional but Recommended)**

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
