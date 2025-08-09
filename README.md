# Datadog Node Monitoring & Auto-Remediation Stack

Monitor, remediate, and govern your blockchain node fleet with Datadog, OPA, and GitHub Copilot.  
Supports Docker, Kubernetes, multi-chain logic, policy-as-code guardrails, and auto-healing.

---

## 🚀 Features

- **Datadog Agent** deployment (Docker/Kubernetes)
- **Monitors** for peer count, latency, block lag
- **Webhook Listener** (Python Flask) with multi-chain logic
- **Auto-remediation** with OPA policies
- **Extensible** for new chains, cloud, or ChatOps

---

## 🏗 Architecture

```
Datadog Agent → Monitors → Webhook (/remediate) → OPA Policy → Remediation (Docker restart/K8s rollout)
```

---

## 🐳 Quick Start (Docker Compose)

```yaml
version: '3.8'
services:
  datadog-agent:
    image: datadog/agent:latest
    container_name: datadog-agent
    environment:
      - DDAPIKEY=yourdatadogapikeyhere
      - DD_SITE=datadoghq.com
      - DDLOGSENABLED=true
      - DDLOGSCONFIGCONTAINERCOLLECT_ALL=true
      - DDAPMENABLED=true
      - DDCONTAINEREXCLUDE="name:datadog-agent"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /proc:/host/proc:ro
      - /sys/fs/cgroup:/host/sys/fs/cgroup:ro
    ports:
      - "8126:8126"
      - "8125:8125/udp"
    restart: unless-stopped
```

---

## ☸️ Quick Start (Kubernetes Helm)

```bash
helm repo add datadog https://helm.datadoghq.com
helm repo update

helm install datadog-agent datadog/datadog \
  --set datadog.apiKey=yourdatadogapikeyhere \
  --set datadog.site=datadoghq.com \
  --set datadog.logs.enabled=true \
  --set datadog.apm.enabled=true \
  --set datadog.processAgent.enabled=true \
  --set agents.containerLogs.enabled=true
```

---

## 📊 Datadog Monitors (Examples)

- **Peer Count Low:**  
  `avg(last5m):min:blockchain.peercount{chain:ethereum,env:prod} < 8`
- **RPC Latency High:**  
  `avg(last_5m):avg:rpc.latency.ms{chain:ethereum,env:prod} > 300`
- **Block Height Lag:**  
  `avg(last5m):max:blockchain.blocklag{chain:ethereum,env:prod} > 5`

All monitors trigger a webhook to `/remediate`.

---

## 🔗 Webhook Listener (Python Flask)

```python
from flask import Flask, request
import subprocess
import requests

app = Flask(__name__)

# ... node_map, fallback_map as per main code ...

@app.route("/remediate", methods=["POST"])
def remediate():
    # Parse alert & tags, call OPA, restart node if allowed
    # See full code in repo
```

---

## 🛡️ OPA Policy (Rego)

```rego
package remediation

allow {
  input.env == "prod"
  input.chain in ["ethereum", "solana", "avax"]
  input.action == "restart"
}
```

---

## 🧪 Testing & CI

- Lint and test Flask webhook (see `.github/workflows/ci.yml`)
- Validate OPA policy responses (unit test or curl)
- Simulate Datadog alerts and check end-to-end flow

---

## 🆘 Troubleshooting

- Ensure Datadog API key and site are correct
- Flask must be reachable by Datadog webhook
- OPA must be running and accessible
- Check logs for remediation failures

---

## 📦 Extending

- Add new chains in `node_map`/`fallback_map`
- Add new OPA policies for approval workflows
- Integrate with Slack/Discord via ChatOps bot

---

## 🤝 Contributing

PRs, issues, and feature requests welcome!  
See [`docs/`](docs/) for API, architecture, and integration guides.

---

## License

MIT