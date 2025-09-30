# 📡 KubePulse — Kubernetes + Monitoring Microservice

![Python](https://img.shields.io/badge/Python-3.11-blue.svg?logo=python&logoColor=white)
![FastAPI](https://img.shields.io/badge/FastAPI-0.111-009688?logo=fastapi&logoColor=white)
![Prometheus](https://img.shields.io/badge/Prometheus-Metrics-E6522C?logo=prometheus)
![Docker](https://img.shields.io/badge/Docker-Enabled-2496ED?logo=docker)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Ready-326CE5?logo=kubernetes)

**KubePulse** is a small Python FastAPI service designed for Kubernetes monitoring demos:
- `/metrics` exposes Prometheus counters, histograms, gauges, and a summary
- `/probe` performs external HTTP checks and records latency
- `/simulate-latency` helps you shape histograms for demos and HPA tests
- `/enqueue` manipulates a gauge to mimic a job queue
- `/health` and `/ready` support container probes

-----

## 🛠 Tech & Languages

| Layer | Tech | Notes |
|------|------|------|
| Language | **Python 3.11** | Popular for platform tooling and exporters |
| Framework | **FastAPI** | Simple to build APIs with auto docs |
| Metrics | **prometheus_client** | Exposes `/metrics` |
| HTTP Client | **httpx** | Async external probes |
| Container | **Docker** | Builds and runs identically everywhere |
| Orchestrator | **Kubernetes** | Deployment, Service, probes |
| Optional | **ServiceMonitor** | For Prometheus Operator setups |

---

## 🌐 Architecture

<p align="center">
  <img src="https://raw.githubusercontent.com/your-org/kubepulse-assets/main/architecture.png" alt="KubePulse Architecture" width="650" />
</p>

Flow:
1. Clients call `/health`, `/ready`, `/probe`, `/simulate-latency`, `/metrics`
2. Middleware exports request metrics
3. Prometheus scrapes `/metrics`
4. Deploy as a Docker container behind a Kubernetes Service
5. Optionally use a **ServiceMonitor** for Prometheus Operator

---

## 📦 Repository Structure

kubepulse/
├─ app/
│ └─ main.py
├─ tests/
│ └─ test_basic.py
├─ k8s/
│ ├─ deployment.yaml
│ ├─ service.yaml
│ └─ servicemonitor.yaml # optional
├─ Dockerfile
├─ requirements.txt
├─ Makefile
└─ README.md

yaml
Copy code

---

## ▶️ Run in Google Colab

Use the one-cell setup in your notebook. After it prints the URL, test:

```python
import httpx
print(httpx.get("http://127.0.0.1:8002/health").json())
print(httpx.get("http://127.0.0.1:8002/metrics").status_code)
🔗 API Endpoints
Method	Path	Description
GET	/	Service info
GET	/health	Liveness check
GET	/ready	Readiness check
GET	/metrics	Prometheus metrics
POST	/probe	Probe external HTTP target
POST	/simulate-latency	Sleep N ms to shape histograms
POST	/enqueue?n=1	Increase queue depth gauge

Examples
Probe an external target

bash
Copy code
curl -X POST http://localhost:8002/probe \
  -H "Content-Type: application/json" \
  -d '{"url":"https://httpbin.org/status/200","timeout":3,"expect_status":200}'
Response:

json
Copy code
{ "target": "https://httpbin.org/status/200", "status_code": 200, "ok": true, "latency_seconds": 0.1234 }
Simulate latency

bash
Copy code
curl -X POST http://localhost:8002/simulate-latency \
  -H "Content-Type: application/json" \
  -d '{"milliseconds": 500}'
Increase queue depth

bash
Copy code
curl -X POST "http://localhost:8002/enqueue?n=5"
📊 Prometheus Metrics
You will see series like:

pgsql
Copy code
kubepulse_http_requests_total{method="GET",path="/health",status="200"} 8
kubepulse_http_request_duration_seconds_bucket{method="POST",path="/simulate-latency",le="0.25"} 3
kubepulse_external_probe_latency_seconds_sum{target="https://httpbin.org/status/200"} 0.527
kubepulse_task_queue_depth 4
kubepulse_up 1
Scrape config (plain Prometheus)
yaml
Copy code
- job_name: "kubepulse"
  static_configs:
    - targets: ["kubepulse.default.svc.cluster.local:80"]
Prometheus Operator (ServiceMonitor)
If you use kube-prometheus-stack:

bash
Copy code
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/servicemonitor.yaml
Make sure the release: prometheus label in ServiceMonitor matches your operator’s label.

🧪 Tests
bash
Copy code
pytest -q
Included tests:

/health and /ready responses

queue and latency path sanity checks

🐳 Docker
Build:

bash
Copy code
docker build -t kubepulse:latest .
Run:

bash
Copy code
docker run -p 8002:8002 kubepulse:latest
Open: http://localhost:8002/health

☸️ Kubernetes
Update image in k8s/deployment.yaml:

yaml
Copy code
image: ghcr.io/YOUR_GH_USERNAME/kubepulse:latest
Apply:

bash
Copy code
kubectl apply -f k8s/
kubectl get pods -l app=kubepulse
kubectl port-forward svc/kubepulse 8002:80
📈 Interactive Docs
Swagger UI: /docs

ReDoc: /redoc

Add screenshots to your repo to make the README visual:

<p align="center"> <img src="https://raw.githubusercontent.com/your-org/kubepulse-assets/main/fastapi-docs.png" alt="FastAPI Docs" width="720" /> </p>
🔐 Production Notes
Add auth (API key or JWT) if the service is public

Scrape through an Ingress with TLS for production

Centralize logs and add tracing if needed

Use HPA on CPU and requests per second when you demo /simulate-latency

👤 Author
Siddharth Raut — DevOps & Cloud Engineer
