# Intelligent Fog Orchestration System

A distributed intelligent fog orchestration system for managing IoT and edge workloads through MQTT communication, Docker-based deployment, telemetry monitoring, token-protected control, desired-state reconciliation, remote node connectivity, and AI-assisted operational support.

This repository is the main entry point for the complete Intelligent Fog Orchestration System. It connects the component repositories, Docker images, deployment flow, workload configuration, project evidence, research paper, and presentation into one organized project package.

The system is built around a controller-node model. The Fog Controller defines the expected workload state for each fog node. Each IoT Smart Node connects to the MQTT broker, receives desired-state updates, manages its local Docker runtime, publishes telemetry, and reconciles mismatches between the expected state and the actual container state.

This is not only a dashboard. It is a distributed fog orchestration platform that combines MQTT communication, Dockerized workload orchestration, telemetry, desired-state reconciliation, heterogeneous edge workloads, AI-assisted operational summaries, token-based control, and remote multi-device deployment.

---

## 1. System Overview

The Intelligent Fog Orchestration System manages workloads across multiple IoT and fog nodes from one central controller.

The system supports same-machine deployment, same-network distributed deployment, and remote multi-device deployment through Tailscale. Because the MQTT broker host is configurable, nodes can connect locally, through LAN IP addresses, or through Tailscale IPs between remote devices.

Local development can use:

```bash
MQTT_HOST=host.docker.internal
```

Remote deployment through Tailscale can use:

```bash
MQTT_HOST=100.x.x.x
```

This allows fog nodes to run across multiple physical machines while remaining under one orchestration layer. One node can run on macOS, another on Linux, another on a Proxmox-hosted machine, while all remain connected to the same MQTT broker through Tailscale networking.

The core behavior of the system is the control loop. The controller stores the desired workload state, the nodes observe their actual Docker runtime state, and the node agents reconcile mismatches automatically.

---

## 2. Architecture

The system has four main components.

The Fog Controller Dashboard is the central orchestration interface. It stores desired state, displays telemetry, shows node status, records events, and generates AI-assisted operational summaries.

The MQTT Broker is the communication backbone between the controller and the fog nodes.

The IoT Smart Node runs on each fog node. It connects to MQTT, receives desired-state updates, manages the local Docker runtime, deploys and stops workloads, publishes telemetry, performs desired-state reconciliation, and supports runtime monitoring.

The Fog Intelligence LLM Service connects to local Ollama models and generates operational summaries for the controller. This service is separate from the Integral Calculator explanation feature.

```text
+------------------------------------------------------+
|                Fog Controller Dashboard              |
|------------------------------------------------------|
| Desired State | Node Status | Event Log | AI Summary |
+-----------------------------+------------------------+
                              |
                              | MQTT
                              |
+-----------------------------+------------------------+
|                       MQTT Broker                    |
+-----------------------------+------------------------+
              |                                |
              | MQTT                           | MQTT
              v                                v
+---------------------------+      +---------------------------+
|    IoT Smart Node 1       |      |      IoT SmartNode 2      |
|---------------------------|      |---------------------------|
| Node Agent                |      | Node Agent                |
| Docker Engine             |      | Docker Engine             |
| Desired-State Loop        |      | Desired-State Loop        |
| Telemetry Publisher       |      | Telemetry Publisher       |
|---------------------------|      |---------------------------|
| Workloads:                |      | Workloads:                |
| - ProMatch                |      | - Integral Calculator     |
| - FluidLab CFD            |      | - CNN Edge Classifier     |
+---------------------------+      +---------------------------+
```

A visual architecture diagram will be stored at:

```text
docs/images/architecture.png
```

---

## 3. Real Node Configuration

The project runtime uses two fog nodes.

Node 1 has the node ID:

```text
node-1
```

Node 1 runs ProMatch and FluidLab CFD. This node represents heavier edge workloads and multi-service orchestration on one fog node.

Node 2 has the node ID:

```text
node-2
```

Node 2 runs the Integral Calculator and CNN Edge Classifier. This node represents symbolic computation, AI-assisted mathematical explanation, and CNN-based edge inference.

The desired state used by the controller is:

```text
node-1:
  - promatch
  - fluidlab

node-2:
  - integral-calculator
  - cnn-edge-classifier
```

After the desired state is applied, the IoT Smart Nodes reconcile their local Docker runtime and start the required workloads.

---

## 4. Component Repositories

This repository documents the complete system. The implementation is split across component repositories.

### Fog Controller

```text
https://github.com/IrfanUruchi/fog-controller
```

Docker image:

```text
irfanuruchi/fog-controller:latest
```

The Fog Controller provides the dashboard and orchestration interface. It stores desired state, displays telemetry, shows events, tracks node status, exposes workload controls, and generates AI-assisted operational summaries.

### IoT Smart Node

```text
https://github.com/IrfanUruchi/iot-smart-node
```

Docker image:

```text
irfanuruchi/iot-smart-node:latest
```

The IoT Smart Node is the node-side orchestration agent. It connects to MQTT, receives desired-state updates, manages local Docker workloads, publishes telemetry, performs desired-state reconciliation, supports self-healing behavior, and applies token-protected command handling.

### Fog Intelligence LLM Service

```text
https://github.com/IrfanUruchi/fog-intelligence-llm
```

Docker image:

```text
irfanuruchi/fog-intelligence-llm:latest
```

The Fog Intelligence LLM Service connects to local Ollama models and generates operational summaries for the controller. It is used for controller-level intelligence and is separate from the Integral Calculator explanation pipeline.

### CNN Edge Classifier

```text
https://github.com/IrfanUruchi/cnn-edge-classifier
```

Docker image:

```text
irfanuruchi/cnn-edge-classifier:latest
```

The CNN Edge Classifier is a MobileNetV2-based computer vision inference workload with a drag-and-drop UI and prediction API. It demonstrates CNN/DCNN inference as a containerized fog workload.

---

## 5. Docker Images

Main system images:

```text
irfanuruchi/fog-controller:latest
irfanuruchi/iot-smart-node:latest
irfanuruchi/fog-intelligence-llm:latest
```

Workload images:

```text
irfanuruchi/promatch:latest
irfanuruchi/fluid-lab-cfd:cpu
irfanuruchi/integral-calculator:latest
irfanuruchi/cnn-edge-classifier:latest
```

The deployable images support:

```text
linux/amd64
linux/arm64
```

---

## 6. Workloads

### ProMatch

ProMatch is a Prolog-based matchmaking workload. It demonstrates that the orchestration system can manage heterogeneous intelligent workloads instead of only standard web services.

### FluidLab CFD

FluidLab CFD is a scientific simulation workload. The CPU image used in the project runtime is:

```text
irfanuruchi/fluid-lab-cfd:cpu
```

A CUDA image also exists separately for GPU-capable environments. In the project setup, FluidLab CFD demonstrates distributed scientific workload execution at the fog layer.

### Integral Calculator

The Integral Calculator is a Haskell-based symbolic computation engine. It supports parsing, simplification, differentiation, symbolic integration, numerical integration, plotting, and AI-assisted explanations.

The image used by the system is:

```text
irfanuruchi/integral-calculator:latest
```

The Integral Calculator independently uses Cerebras-hosted inference for explanation generation. Because of this, `CEREBRAS_API_KEY` is injected into `node-2`, not into the Fog Intelligence LLM Service.

### CNN Edge Classifier

The CNN Edge Classifier is a lightweight computer vision inference workload using MobileNetV2 with ImageNet pretrained weights. It runs as a Dockerized fog workload with both a web UI and a prediction API.

The service accepts image uploads, performs CNN-based classification, and returns the predicted class, top-5 predictions, confidence scores, and inference time.

The image used by the system is:

```text
irfanuruchi/cnn-edge-classifier:latest
```

This workload demonstrates that the same MQTT and desired-state orchestration model can deploy CNN/DCNN inference workloads at the edge, alongside Prolog reasoning, CFD simulation, and symbolic computation services.

---

## 7. Environment Variables

The MQTT broker host is configurable so the system can run locally, across a LAN, or across remote devices connected through Tailscale.

Local Docker Desktop setup:

```bash
MQTT_HOST=host.docker.internal
MQTT_PORT=1883
```

LAN setup:

```bash
MQTT_HOST=192.168.x.x
MQTT_PORT=1883
```

Remote Tailscale setup:

```bash
MQTT_HOST=100.x.x.x
MQTT_PORT=1883
```

Node 1 example:

```bash
NODE_ID=node-1
MQTT_HOST=host.docker.internal
MQTT_PORT=1883
DEVICE_TOKEN=changeme
```

Node 2 example:

```bash
NODE_ID=node-2
MQTT_HOST=host.docker.internal
MQTT_PORT=1883
DEVICE_TOKEN=changeme
CEREBRAS_API_KEY=your_api_key_here
```

Fog Intelligence LLM Service example:

```bash
OLLAMA_BASE_URL=http://host.docker.internal:11434
OLLAMA_MODEL=llama3.2
```

`CEREBRAS_API_KEY` is used only by the Integral Calculator workload. The Fog Intelligence LLM Service does not use Cerebras; it connects to local Ollama through `OLLAMA_BASE_URL` and `OLLAMA_MODEL`.

Runtime secrets, API keys, and tokens are provided through environment variables or local `.env` files. Local `.env` files are excluded from version control.

---

## 8. Running the System Locally

The following commands start the full system on one development machine.

### Step 1: Start the MQTT Broker

```bash
docker run -d \
  --name mqtt-broker \
  -p 1883:1883 \
  eclipse-mosquitto
```

### Step 2: Start the Fog Intelligence LLM Service

The Fog Intelligence LLM Service connects to Ollama and provides operational summaries for the controller.

```bash
docker run -d \
  --name fog-intelligence-llm \
  -p 8001:8001 \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 \
  -e OLLAMA_MODEL=llama3.2 \
  irfanuruchi/fog-intelligence-llm:latest
```

### Step 3: Start the Fog Controller

```bash
docker run -d \
  --name fog-controller \
  -p 8001:8001 \
  -e MQTT_HOST=host.docker.internal \
  -e MQTT_PORT=1883 \
  -e DEVICE_TOKEN=changeme \
  -e FOG_INTELLIGENCE_URL=http://host.docker.internal:8001 \
  irfanuruchi/fog-controller:latest
```

Open the controller dashboard:

```text
http://localhost:8001
```

### Step 4: Start IoT Smart Node 1

```bash
docker run -d \
  --name node-1 \
  -e NODE_ID=node-1 \
  -e MQTT_HOST=host.docker.internal \
  -e MQTT_PORT=1883 \
  -e DEVICE_TOKEN=changeme \
  -v /var/run/docker.sock:/var/run/docker.sock \
  irfanuruchi/iot-smart-node:latest
```

### Step 5: Start IoT Smart Node 2

```bash
docker run -d \
  --name node-2 \
  -e NODE_ID=node-2 \
  -e MQTT_HOST=host.docker.internal \
  -e MQTT_PORT=1883 \
  -e DEVICE_TOKEN=changeme \
  -e CEREBRAS_API_KEY=$CEREBRAS_API_KEY \
  -v /var/run/docker.sock:/var/run/docker.sock \
  irfanuruchi/iot-smart-node:latest
```

The Docker socket mount is required because the IoT Smart Node directly manages the host Docker runtime.

```bash
-v /var/run/docker.sock:/var/run/docker.sock
```

After both nodes start, the dashboard should show `node-1` and `node-2` as connected.

---

## 9. Distributed Deployment with Tailscale

The system can be deployed across multiple physical machines by connecting them through Tailscale.

In this setup, the MQTT broker runs on one device, and the Fog Controller plus IoT Smart Nodes connect to the broker through its Tailscale IP address.

Example MQTT host:

```bash
MQTT_HOST=100.x.x.x
```

Node 1 on a remote machine can be started with:

```bash
docker run -d \
  --name node-1 \
  -e NODE_ID=node-1 \
  -e MQTT_HOST=100.x.x.x \
  -e MQTT_PORT=1883 \
  -e DEVICE_TOKEN=changeme \
  -v /var/run/docker.sock:/var/run/docker.sock \
  irfanuruchi/iot-smart-node:latest
```

Node 2 on another remote machine can be started with:

```bash
docker run -d \
  --name node-2 \
  -e NODE_ID=node-2 \
  -e MQTT_HOST=100.x.x.x \
  -e MQTT_PORT=1883 \
  -e DEVICE_TOKEN=changeme \
  -e CEREBRAS_API_KEY=$CEREBRAS_API_KEY \
  -v /var/run/docker.sock:/var/run/docker.sock \
  irfanuruchi/iot-smart-node:latest
```

This allows the project to run across macOS, Linux, Proxmox-hosted machines, and other Docker-capable devices while still being controlled by the same Fog Controller.

---

## 10. Desired-State Runtime

The desired state used by the project is:

```text
node-1:
  - promatch
  - fluidlab

node-2:
  - integral-calculator
  - cnn-edge-classifier
```

The controller publishes this expected state through MQTT. Each IoT Smart Node receives the desired-state update, checks its local Docker runtime, and starts or stops workloads to match the expected configuration.

The event log records the orchestration behavior, including deployment actions, workload state changes, and reconciliation events.

---

## 11. Workload URLs

After deployment, the workloads are available through their exposed ports.

```text
ProMatch:              http://localhost:3000
FluidLab CFD:          http://localhost:8082
Integral Calculator:   http://localhost:5050
CNN Edge Classifier:   http://localhost:8600/ui
```

In distributed deployments, replace `localhost` with the node machine IP address, LAN address, or Tailscale IP.

---

## 12. Multi-Architecture Docker Publishing

Each deployable image is published for both `linux/amd64` and `linux/arm64`.

Build command:

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t irfanuruchi/<image-name>:latest \
  --push .
```

Verify the published image manifests:

```bash
docker buildx imagetools inspect irfanuruchi/fog-controller:latest
docker buildx imagetools inspect irfanuruchi/iot-smart-node:latest
docker buildx imagetools inspect irfanuruchi/fog-intelligence-llm:latest
docker buildx imagetools inspect irfanuruchi/cnn-edge-classifier:latest
```

Expected platforms:

```text
linux/amd64
linux/arm64
```

---

## 13. Reliability Model

The reliability model is based on desired-state reconciliation.

The Fog Controller stores the expected workload state for each node. Each IoT Smart Node checks the local Docker runtime and compares it with the desired state received from the controller. When the actual state does not match the expected state, the node agent performs the required action.

Example:

```text
Desired state:
node-1 should run promatch and fluidlab

Actual state:
fluidlab is not running

Reconciliation action:
node-1 starts fluidlab again
```

This avoids relying only on one-time commands. The node continuously observes runtime state and moves the system back toward the expected configuration.

---

## 14. Security Model

The system includes token-based protection for controlled access paths and keeps runtime secrets outside the repository. API keys, service tokens, and runtime configuration values are passed through environment variables or local `.env` files instead of being hardcoded into the source code.

The implementation also includes rate limiting, burst limiting, restricted configuration updates, Docker-managed labels, and controlled workload handling so that containers created by the orchestration system can be identified and managed safely by the node agent.

The IoT Smart Node requires access to the Docker socket because it manages the local Docker runtime directly. This gives the node agent control over workload deployment on its host machine, so the node agent is treated as a trusted runtime component.

MQTT is used as the communication backbone between the controller and the nodes. The system separates node identity, telemetry, desired-state messages, and orchestration events through structured message handling.

The current implementation focuses on practical token usage, secret separation, runtime configuration, repository hygiene, and controlled orchestration behavior.

---

## 15. Evaluation

The system can be evaluated by running the full project setup and checking that the controller receives telemetry, both nodes appear online, workloads are deployed through Docker, desired-state configuration is applied, different workloads run on different nodes, reconciliation works after a mismatch, the Integral Calculator generates an AI-assisted explanation, the CNN Edge Classifier performs image classification, and the Fog Controller generates an operational summary through the Fog Intelligence LLM Service.

The event log is the main runtime evidence because it shows the orchestration process rather than only the final UI state.

---

## 16. Project Evidence

Screenshots and diagrams are stored under:

```text
docs/images/
```

Expected project evidence includes:

```text
architecture.png
dashboard.png
node-1-promatch-fluidlab.png
node-2-integral-cnn.png
event-log-ai-summary.png
docker-hub-multiarch.png
```

These images document the system architecture, controller state, distributed nodes, deployed workloads, AI-assisted explanation output, CNN inference output, operational summaries, orchestration events, and Docker Hub multi-architecture image support.

---

## 17. Project Structure

```text
intelligent-fog-orchestration-system/
├── README.md
├── LICENSE
├── docs/
│   ├── architecture.md
│   ├── execution-flow.md
│   ├── troubleshooting.md
│   └── images/
│       ├── architecture.png
│       ├── dashboard.png
│       ├── node-1-promatch-fluidlab.png
│       ├── node-2-integral-cnn.png
│       ├── event-log-ai-summary.png
│       └── docker-hub-multiarch.png
├── paper/
│   └── Intelligent-Fog-Orchestration-System.pdf
├── presentation/
│   └── Intelligent-Fog-Orchestration-System.pptx
└── scripts/
    └── project-run-checklist.md
```

---

## 18. Troubleshooting

If the MQTT broker is not reachable, check that the broker container is running.

```bash
docker ps
docker logs mqtt-broker
```

If the controller or nodes are running through Docker Desktop, use `host.docker.internal` as the broker hostname. If the system is distributed across a LAN, use the broker machine LAN IP. If the system is distributed through Tailscale, use the broker machine Tailscale IP.

If a node does not appear online, check the node logs.

```bash
docker logs node-1
docker logs node-2
```

If a workload does not deploy, confirm that the Docker socket is mounted into the IoT Smart Node container.

```bash
-v /var/run/docker.sock:/var/run/docker.sock
```

If the controller operational summary does not work, check that the Fog Intelligence LLM Service is running and that Ollama is reachable.

```bash
docker ps
docker logs fog-intelligence-llm
```

If the Integral Calculator explanation does not work, check that `CEREBRAS_API_KEY` is available in `node-2`.

```bash
docker logs node-2
```

If the dashboard does not open, check the controller logs and confirm that the controller port is mapped.

```bash
docker logs fog-controller
```

---

## 19. Academic Deliverables

The final project package includes this main repository, the component repositories, Docker images, project screenshots, the research paper, and the presentation.

The paper describes the architecture, implementation, fog deployment model, intelligent support layer, workload case studies, evaluation, security model, limitations, and future work.

The presentation focuses on the system goal, architecture, MQTT communication, node design, Docker orchestration, AI-assisted support, distributed deployment, reliability behavior, workload case studies, evaluation, and execution flow.

---

## 20. Future Work

Future versions can extend the platform with more advanced workload placement, persistent event storage, deeper resource telemetry, automatic workload migration, richer scheduling policies, and larger multi-node deployment scenarios.

The current version focuses on the complete working system: Fog Controller, MQTT communication, IoT Smart Nodes, Docker workload management, telemetry, desired-state reconciliation, remote node support through Tailscale, heterogeneous workloads, CNN edge inference, and AI-assisted operational support.

---

## 21. License

This project is released under the MIT License.

---

## 22. Author

**Irfan Uruçi**
Computer Engineering / Computer Science
South East European University
Intelligent Systems Course Project
Academic Year 2026
