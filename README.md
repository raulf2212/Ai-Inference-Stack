# Local AI Inference Stack with Podman
An automated CI/CD pipeline template using Jenkins and Podman to deploy a local, monitorable AI inference stack running on Rocky Linux. For the backend the project runs a Qwen 2.5 AI Model using llama.cpp and for the frontend it uses Open WebUI for a simple interface. For the monitoring: Node Exporter, Prometheus and Grafana are used to track performance, CPU, memory usage, etc. By using OCI-compliant containers managed by Podman, all components communicate inside an isolated virtual bridge network (llm-net).

This project was created as part of a collaboration UVT x IBM MLOps for the Team Project subject.

# Stack Architecture

       ┌────────────────────────────────────────────────────────┐
       │               llm-net (bridge network)                 │
       │                                                        │
       │  ┌──────────────┐              ┌──────────────┐        │
       │  │ llama-server │ ◄──────────  │  open-webui  │        │
       │  │  port 8080   │              │  port 3000   │        │
       │  └──────┬───────┘              └──────────────┘        │
       │         │                                              │
       │         ▼                                              │
       │  ┌──────────────┐              ┌──────────────┐        │
       │  │node-exporter │ ◄──────────  │  prometheus  │        │
       │  │  port 9100   │              │  port 9090   │        │
       │  └──────────────┘              └──────┬───────┘        │
       │                                       │                │
       │                                       ▼                │
       │                                ┌──────────────┐        │
       │                                │   grafana    │        │
       │                                │  port 3001   │        │
       │                                └──────────────┘        │
       └────────────────────────────────────────────────────────┘

# Port Mapping

| Service | Port | Access URL | Description |
| :--- | :--- | :--- | :--- |
| **Open WebUI** | `3000` | `http://localhost:3000` | Main chat interface |
| **Grafana** | `3001` | `http://localhost:3001` | Visual monitoring dashboards (Login: admin/admin) |
| **Llama Server** | `8080` | `http://localhost:8080` | Raw AI inference engine API backend |
| **Prometheus** | `9090` | `http://localhost:9090` | Time-series metrics database |

# WebUi Interface

<img width="1835" height="873" alt="Screenshot 2026-06-08 163115" src="https://github.com/user-attachments/assets/dd6f71bc-2745-456b-bd9d-2b967c0c74f2" />

# Grafana Dashboard

<img width="1580" height="843" alt="Screenshot 2026-06-08 163326" src="https://github.com/user-attachments/assets/b9f15288-aabe-47b9-9bb6-b176e7054c2a" />

# Deploying the project

### Prerequisites

- OS: Rocky Linux 9 environment.
- Runtimes: Podman engine and active Jenkins node.
- Permissions: Ensure user jenkins is properly whitelisted in /etc/sudoers to issue sub-level execution orders (sudo podman).

### Running

1. Open the VM Ports by opening the Rocky Linux terminal and running: 
sudo firewall-cmd --add-port={3000,3001,8080,9090}/tcp --permanent && sudo firewall-cmd --reload
2. Go to your local Jenkins interface (http://localhost:8081).
3. Create a new project and insert the whole Jenkinsfile into Pipeline Definition > Pipeline Script and click Build Now.
4. Go to http://localhost:3000 and start chatting with the AI Model.

### Setting up the Grafana Dashboard

1. Go to http://localhost:3001 (Username: admin | Password: admin).
2. Go to Connections > Data sources > Add Prometheus.
3. Set the Prometheus server URL endpoint to: http://prometheus:9090
4. Save and import your custom metric layouts to track resource consumption.
