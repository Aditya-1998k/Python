### From Stress to Success: Load Testing Python Apps & Visualizing Performance
Speaker: [Allen Y](https://www.linkedin.com/in/allen-yesudasan/)

**Introduction & Problem Statement**
Why Load Testing?
1. Applications often work fine with few users but can fail under real-world traffic.
2. Detect bottlenecks before production → saves cost, improves user experience.
3. Helps in capacity planning, resilience checks, and scaling strategies.

Common performance issues identified by load testing:
1. Latency spikes
2. Memory leaks
3. Database query inefficiencies
4. Poor caching strategies
5. Thread/connection pool exhaustion

**Anatomy of Load Testing**
Goal: Simulate real user behavior and measure how the system performs.

Load testing types covered:
1. `Stress Testing`: Push system beyond normal load until it breaks → find breaking point.
2. `Soak Testing`: Long-duration load to detect memory leaks, stability issues.
3. `Spike Testing`: Sudden traffic surge → resilience check.
4. `Acceptance Load Test`: Ensure system meets defined SLA/SLO under expected traffic.
5. `Exploratory Load Test`: Ad-hoc testing to discover unexpected bottlenecks.

**Tools Overview**

**Locust**
1. A Python-based load testing tool.
2. Developer-friendly: write tests as Python scripts, not XML (like JMeter).
3. Simulates user behavior with code (tasks, weight, randomization).
4. Provides web UI to start/stop tests, view live results.

Comparison with Other Tools:
1. JMeter → XML-heavy, less developer friendly.
2. k6 → Modern, scalable, but requires JavaScript.
3. WebLoad / Gatling → Enterprise-oriented, less Python ecosystem friendly.
4. Locust → Python-native, easy to extend, open source, strong community.

**Prometheus**
1. `Time series database` → optimized for monitoring and alerting.
2. Works by scraping metrics from applications (usually via /metrics endpoint over HTTP).

Metrics Types:
1. Counter → only increases (e.g., total requests).
2. Gauge → goes up and down (e.g., current memory usage).
3. Histogram/Summary → distribution of values (e.g., request latency).

**Grafana**
1. Visualization and monitoring tool.
2. Queries data from Prometheus → displays dashboards in real time.
3. Useful for creating actionable dashboards:
4. Requests per second
5. Error rate (%)
6. Latency percentiles (p50, p95, p99)
7. Resource usage (CPU, memory, DB connections)

End-to-End Setup
```
Locust ----> Web App
   |
   | (exposes /metrics endpoint)
   v
Locust Exporter --> Prometheus (scrapes metrics)
   |
   v
Query data ----> Grafana (dashboards & visualization)
```

**Steps in Practice**:
1. Install Locust → pip install locust
2. Write test script simulating user flows.
3. Run Locust to generate load against the app.
4. Expose /metrics endpoint in the app or via Locust exporter.
5. Configure Prometheus to scrape data from Locust + app endpoints.
6. Connect Grafana to Prometheus → build dashboards.
7. Run load tests and monitor dashboards in real time.

Let's do that:
```
loadtest-demo/
│── docker-compose.yml
│── locust/
│   ├── Dockerfile
│   ├── locustfile.py
│── webapp/
│   ├── Dockerfile
│   ├── app.py
│── prometheus/
│   ├── prometheus.yml
```

**webapp/app.py**
```python
from flask import Flask
import time, random

app = Flask(__name__)

@app.route("/")
def index():
    return "Hello from Demo App!"

@app.route("/about")
def about():
    time.sleep(random.uniform(0.1, 0.3))  # simulate latency
    return "About Page"

@app.route("/products")
def products():
    time.sleep(random.uniform(0.2, 0.5))  # simulate heavier page
    return "Products Page"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```
**webapp/Dockerfile**
```Dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY app.py .
RUN pip install flask
CMD ["python", "app.py"]
```
**locust/locustfile.py**
```python
from locust import HttpUser, task, between

class DemoUser(HttpUser):
    wait_time = between(1, 3)

    @task
    def index(self):
        self.client.get("/")

    @task
    def about(self):
        self.client.get("/about")

    @task
    def products(self):
        self.client.get("/products")

    @task
    def contact(self):
        self.client.get("/contact")
```

**locust/Dockerfile**
```Dockerfile
FROM python:3.11-slim
WORKDIR /locust
COPY locustfile.py .
RUN pip install locust
CMD ["locust", "-f", "locustfile.py", "--host", "http://webapp:5000"]
```

**prometheus/prometheus.yml**
```yaml
global:
  scrape_interval: 5s

scrape_configs:
  - job_name: "locust"
    static_configs:
      - targets: ["locust:8089"]
```
**docker-compose.yml**
```yaml
version: "3.9"
services:
  webapp:
    build: ./webapp
    container_name: demo-webapp
    ports:
      - "5000:5000"

  locust:
    build: ./locust
    container_name: demo-locust
    ports:
      - "8089:8089"   # Locust Web UI
    depends_on:
      - webapp

  prometheus:
    image: prom/prometheus
    container_name: demo-prometheus
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    depends_on:
      - locust

  grafana:
    image: grafana/grafana
    container_name: demo-grafana
    ports:
      - "3000:3000"
    depends_on:
      - prometheus
```

Run Everything:
```
docker-compose up --build
```

### Demo Flow
1. Web App → runs at http://localhost:5000
2. Locust Web UI → http://localhost:8089
- Enter number of users (e.g. 1000) and spawn rate.
- Start test against http://webapp:5000.
3. Prometheus → http://localhost:9090
- Query metrics like locust_requests_total.
4. Grafana → http://localhost:3000
- login (admin/admin)
- Add Prometheus as data source (http://prometheus:9090).
- Create panels to show requests/sec, latency, error rate.
