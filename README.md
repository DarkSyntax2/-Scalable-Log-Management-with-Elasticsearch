# Elasticsearch Centralized Logging System

The ELK stack consists of Elasticsearch, Logstash, and Kibana.

They provide a powerful, flexible, and scalable solution for managing and making sense of large amounts of data.

**`Logstash`**: a data processing pipeline which gathers, processes, and forwards data (logs, metrics) from various sources to Elasticsearch.  
**`Elasticsearch`**: a distributed search and analytics engine that stores this processed data and enables powerful search and analytics capabilities.  
**`Kibana`**: a data visualization and exploration tool that provides a graphical interface to explore data in Elasticsearch.

---

## Benefits of ELK Stack

- **Real-time Insights**: Enables immediate visualization and analysis of data.
- **Scalability**: Designed to scale with large datasets and system expansion.
- **Flexibility**: Supports a wide variety of input sources and data types.

---

## Getting Started

Workflow Overview:

- Provision Servers with Terraform  
- Configure Users on Linux Servers  
- Setup ELK Stack on Central Server  
- Enable ELK Clustering  
- Add Remote Hosts  
- Develop Kibana Visualization  
- Create Kibana Dashboard  

Machine image: `CentOS 8`.

---

## Provision Servers with Terraform

Install AWS CLI:

```bash
sudo apt install curl unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin
```

Verify installation:

```bash
aws --version
```

Clone the repository:

```bash
git clone <your_forked_repo_url>
```

Run Terraform commands:

```bash
cd <your_repo>/terraform
terraform init
terraform validate
terraform plan
terraform apply -auto-approve
```

Access EC2 instances via SSH:

```bash
ssh -i private-key/terraform-key.pem ec2-user@<ipaddress>
```

---

## User Configuration on Linux Servers

Add a new admin user:

```bash
sudo passwd
sudo useradd adminuser
sudo usermod -aG wheel adminuser
echo "adminuser ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/adminuser
sudo chmod 0440 /etc/sudoers.d/adminuser
sudo chown root:root /etc/sudoers.d/adminuser
su - adminuser
sudo ls -la /root
```

Secure SSH access:

```bash
sudo vi /etc/ssh/sshd_config
# Set PermitRootLogin no
sudo systemctl restart sshd
sudo grep PermitRootLogin /etc/ssh/sshd_config
```

---

## ELK Stack Setup

### Elasticsearch

Install Elasticsearch:

```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.13.4-x86_64.rpm
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.13.4-x86_64.rpm.sha512
shasum -a 512 -c elasticsearch-8.13.4-x86_64.rpm.sha512
sudo rpm --install elasticsearch-8.13.4-x86_64.rpm
```

Configure cluster:

```bash
sudo vi /etc/elasticsearch/elasticsearch.yml
```

Example config:
```yaml
cluster.name: syslog
node.name: central-server-1
network.host: [10.33.10.1, _local_]
discovery.zen.ping.unicast.hosts: ["10.33.10.1", "10.33.10.6"]
```

Start service:

```bash
sudo systemctl daemon-reload
sudo systemctl start elasticsearch.service
sudo systemctl enable elasticsearch.service
```

---

### Logstash

Install Logstash:

```bash
sudo yum install -y java-1.8.0-openjdk
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

Configure Logstash repository:

```bash
sudo vi /etc/yum.repos.d/logstash.repo
```

Add repo content and install:

```bash
sudo yum install logstash
```

Create config file:

```bash
sudo vi /etc/logstash/conf.d/syslog.conf
```

Use the same input/filter/output blocks from the original.

Start service:

```bash
sudo systemctl start logstash.service
sudo systemctl enable logstash.service
```

---

### Kibana

Install and configure Kibana:

```bash
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.13.4-x86_64.rpm
shasum -a 512 -c kibana-8.13.4-x86_64.rpm.sha512
sudo rpm --install kibana-8.13.4-x86_64.rpm
sudo vi /etc/kibana/kibana.yml
```

Add:
```yaml
server.host: "10.33.10.1"
```

Start Kibana:

```bash
sudo systemctl start kibana.service
sudo systemctl enable kibana.service
```

---

## Enable ELK Clustering

On primary node:

```bash
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s node
```

On secondary node (`central-server-2`):

Repeat installation and use the token:
```bash
/usr/share/elasticsearch/bin/elasticsearch-reconfigure-node --enrollment-token <token>
```

---

## Add Remote Hosts & Ansible

Install Ansible:

```bash
sudo yum install epel-release
sudo yum install ansible
```

Replace hardcoded usernames with:
- `adminuser`
- Generic IPs or node labels

Configure Ansible Vault generically:

```bash
ansible-vault create /path/to/values.yml --vault-password-file=/path/to/secret-vault.pass
```
