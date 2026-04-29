# elastic-security-lab

![Elasticsearch](https://img.shields.io/badge/Elasticsearch-8.17-blue)
![Kibana](https://img.shields.io/badge/Kibana-8.17-yellow)
![Filebeat](https://img.shields.io/badge/Filebeat-8.17-green)
![License](https://img.shields.io/badge/License-MIT-purple)

Elastic Security Lab is a laboratory environment designed for security analysts (SOC / Threat Hunters) to practice log ingestion, analysis, and detection using Elasticsearch, Kibana, and Filebeat.

It includes a functional architecture, training datasets, and a ready-to-use ingestion pipeline for local environments.

---

## Requirements

- Docker
    
- Docker Compose
    
- WSL2 (Windows only)
    
- Git
    

---

## Quick Start

```bash
git clone https://github.com/beathunterzero/elastic-security-lab.git
cd elastic-security-lab
```

### 1. Configure credentials

Before starting, generate a new password for `kibana_system`:

```bash
docker exec -it elasticsearch bin/elasticsearch-reset-password -u kibana_system
```

Then update the corresponding variable in `docker-compose.yml`:

```
ELASTICSEARCH_PASSWORD=<generated_password>
```

---

### 2. Prepare dataset structure

```bash
mkdir -p datasets/windows datasets/linux datasets/aws datasets/azure datasets/firewall
```

---

### 3. Start the lab

```bash
docker-compose up -d
```

Default access:

```
http://localhost:5601
```

Initial credentials:

```
username: elastic
password: changeme
```

---

## Lab Usage

### Log ingestion

Place files in the corresponding paths:

```
datasets/windows/
datasets/linux/
datasets/aws/
datasets/azure/
```

Filebeat will automatically process the logs and send them to Elasticsearch.

---

### Data Views in Kibana

Create a Data View with the pattern:

```
filebeat-*
```

This enables:

- Discover
    
- Dashboards
    
- Lens
    

---

### User management

Path in Kibana:

```
Stack Management → Security → Users
```

Recommended roles:

- kibana_admin
    
- monitoring_user
    
- viewer
    

---

## Project Structure

```
elastic-security-lab/
│
├── datasets/
│         
├── filebeat/
│   └── filebeat.yml
│   
├── docs/                  
│   ├── architecture/    
│   └── procesos/
│
├── docker-compose.yml
└── README.md
```

---

## Datasets

The lab uses public datasets for training:

- Windows Event Logs
    
- Linux auth logs
    
- AWS CloudTrail / GuardDuty
    
- Azure Activity / Sign-In
    
- Firewall logs
    

---

## Security

This project is intended for local environments and educational purposes.  
It does not include sensitive data or production configurations.

---

## License

MIT


---

## Autor

**beathunterzero**  
Cyber Threat Hunting & Security
