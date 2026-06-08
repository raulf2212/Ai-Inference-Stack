pipeline {
    agent any

    environment {
        JENKINS_NODE_COOKIE = 'dontKillMe'
    }

    stages {
        stage('Create Directories & Permissions') {
            steps {
                sh 'sudo mkdir -p /opt/models /opt/openwebui-data /etc/prometheus'
                sh 'sudo chown -R jenkins:jenkins /opt/models /opt/openwebui-data /etc/prometheus'
            }
        }

        stage('Create podman network') {
            steps {
                sh '''
                    if sudo podman network exists llm-net 2>/dev/null; then
                        echo "Network llm-net already exists, skipping..."
                    else
                        sudo podman network create llm-net
                    fi
                '''
            }
        }

        stage('Download model') {
            steps {
                sh '''
                    if [ -f /opt/models/model.gguf ]; then
                        echo "Model already exists, skipping download..."
                    else
                        curl -L --progress-bar -o /opt/models/model.gguf "https://huggingface.co/Qwen/Qwen2.5-0.5B-Instruct-GGUF/resolve/main/qwen2.5-0.5b-instruct-q4_k_m.gguf"
                    fi
                '''
            }
        }

        stage('Start llama-server container') {
            steps {
                sh '''
                    if sudo podman ps -a --format "{{.Names}}" | grep -q "^llama-server$"; then
                        sudo podman rm -f llama-server
                    fi
                    sudo podman run -d --name llama-server --network llm-net -v /opt/models:/models:Z -p 8080:8080 ghcr.io/ggml-org/llama.cpp:server -m /models/model.gguf --host 0.0.0.0 --port 8080 -t 2
                '''
            }
        }

        stage('Start open-webui container') {
            steps {
                sh '''
                    if sudo podman ps -a --format "{{.Names}}" | grep -q "^open-webui$"; then
                        sudo podman rm -f open-webui
                    fi
                    sudo podman run -d --name open-webui --network llm-net -p 3000:8080 -v /opt/openwebui-data:/app/backend/data:Z -e OPENAI_API_BASE_URL=http://llama-server:8080/v1 -e OPENAI_API_KEY=sk-no-key-required ghcr.io/open-webui/open-webui:main
                '''
            }
        }

        stage('Start Node Exporter') {
            steps {
                sh '''
                    if sudo podman ps -a --format "{{.Names}}" | grep -q "^node-exporter$"; then
                        sudo podman rm -f node-exporter
                    fi
                    sudo podman run -d --name node-exporter --network llm-net -p 9100:9100 --pid host -v /:/host:ro,rslave docker.io/prom/node-exporter:latest --path.rootfs=/host
                '''
            }
        }

        stage('Create Prometheus Config') {
            steps {
                sh '''
                    sudo tee /etc/prometheus/prometheus.yml > /dev/null << 'EOF'
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
EOF
                    sudo chown -R jenkins:jenkins /etc/prometheus
                '''
            }
        }

        stage('Start Prometheus') {
            steps {
                sh '''
                    if sudo podman ps -a --format "{{.Names}}" | grep -q "^prometheus$"; then
                        sudo podman rm -f prometheus
                    fi
                    sudo podman run -d -p 9090:9090 --name prometheus --network llm-net -v /etc/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:Z docker.io/prom/prometheus:latest
                '''
            }
        }

        stage('Start Grafana') {
            steps {
                sh '''
                    if sudo podman ps -a --format "{{.Names}}" | grep -q "^grafana$"; then
                        sudo podman rm -f grafana
                    fi
                    sudo podman run -d -p 3001:3000 --name grafana --network llm-net docker.io/grafana/grafana:latest
                '''
            }
        }
    }
}
