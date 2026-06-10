# Project Run Checklist

Execution checklist for the Intelligent Fog Orchestration System.

This file documents the runtime order, node configuration, workload placement, verification commands, and service endpoints used during the project presentation.

## 1. Runtime Components

```text
Fog Controller
MQTT Broker
IoT Smart Node 1
IoT Smart Node 2
Docker workload containers
Fog Intelligence LLM
Local Ollama runtime
```

## 2. Runtime Architecture

Main orchestration path:

```text
Fog Controller
  -> MQTT Broker
      -> IoT Smart Nodes
          -> Docker workloads
```

Operational summary path:

```text
Fog Controller
  -> Fog Intelligence LLM
      -> local Ollama
```

The Fog Intelligence LLM service is used by the Fog Controller for controller-level operational summaries. It is not responsible for deploying workloads and it does not communicate with the IoT Smart Nodes directly.

## 3. Start MQTT Broker

```bash
docker run -d \
  --name mqtt-broker \
  -p 1883:1883 \
  eclipse-mosquitto
```

Verify:

```bash
docker ps
docker logs mqtt-broker
```

## 4. Start Fog Intelligence LLM

Start the Fog Intelligence LLM service before using the controller summary feature.

```bash
docker run -d \
  --name fog-intelligence-llm \
  -p 8001:8001 \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 \
  -e OLLAMA_MODEL=llama3.2 \
  irfanuruchi/fog-intelligence-llm:latest
```

Verify:

```bash
docker logs fog-intelligence-llm
```

The service connects to local Ollama through `OLLAMA_BASE_URL` and `OLLAMA_MODEL`.

## 5. Start Fog Controller

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

Dashboard:

```text
http://localhost:8001
```

Verify:

```bash
docker logs fog-controller
```

The Fog Controller connects to MQTT for node orchestration and connects to Fog Intelligence LLM for operational summaries.

## 6. Start IoT Smart Node 1

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

Verify:

```bash
docker logs node-1
```

Expected workloads on `node-1`:

```text
promatch
fluidlab
```

## 7. Start IoT Smart Node 2

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

Verify:

```bash
docker logs node-2
```

Expected workloads on `node-2`:

```text
integral-calculator
cnn-edge-classifier
```

`CEREBRAS_API_KEY` is passed to `node-2` because the Integral Calculator workload uses it for explanation generation. Fog Intelligence LLM does not use Cerebras.

## 8. Desired State

The project uses the following desired-state configuration:

```text
node-1:
  - promatch
  - fluidlab

node-2:
  - integral-calculator
  - cnn-edge-classifier
```

The Fog Controller publishes this desired state through MQTT. Each IoT Smart Node compares the expected workload list with the actual Docker runtime and reconciles missing or stopped managed workloads.

## 9. Service Endpoints

Local endpoints:

```text
Fog Controller:         http://localhost:8001
ProMatch:               http://localhost:3000
FluidLab CFD:           http://localhost:8082
Integral Calculator:    http://localhost:5050
CNN Edge Classifier:    http://localhost:8600/ui
```

For LAN or Tailscale deployment, `localhost` is replaced with the IP address of the machine running the service.

## 10. Runtime Verification

Check running containers:

```bash
docker ps
```

Check controller logs:

```bash
docker logs fog-controller
```

Check node logs:

```bash
docker logs node-1
docker logs node-2
```

Check Fog Intelligence LLM logs:

```bash
docker logs fog-intelligence-llm
```

The IoT Smart Node requires Docker socket access because it manages workload containers on the host Docker runtime:

```bash
-v /var/run/docker.sock:/var/run/docker.sock
```

## 11. Multi-Architecture Verification

Published image manifests can be checked with:

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

## 12. Project Evidence

Recommended evidence files:

```text
docs/images/architecture.png
docs/images/dashboard.png
docs/images/node-1-promatch-fluidlab.png
docs/images/node-2-integral-cnn.png
docs/images/event-log-ai-summary.png
docs/images/docker-hub-multiarch.png
```

## 13. Presentation Flow

```text
1. Open the Fog Controller dashboard.
2. Show node-1 and node-2 connected through MQTT.
3. Show the desired-state configuration.
4. Show workload placement on each node.
5. Open ProMatch.
6. Open FluidLab CFD.
7. Open Integral Calculator.
8. Open CNN Edge Classifier.
9. Run an image prediction through the CNN workload.
10. Show telemetry and orchestration events.
11. Show Fog Intelligence LLM operational summary from the controller.
12. Stop one managed workload and show desired-state reconciliation.
```

## 14. Component Responsibilities

```text
Fog Controller:
  dashboard, desired state, MQTT orchestration, telemetry view, event log,
  operational summary requests to Fog Intelligence LLM

MQTT Broker:
  communication layer between Fog Controller and IoT Smart Nodes

IoT Smart Node:
  node-side runtime agent, Docker workload management, telemetry,
  desired-state reconciliation

Fog Intelligence LLM:
  local Ollama-based operational summary service used by the Fog Controller

ProMatch:
  Prolog reasoning workload

FluidLab CFD:
  scientific simulation workload

Integral Calculator:
  Haskell symbolic computation workload with separate Cerebras explanation support

CNN Edge Classifier:
  MobileNetV2 CNN inference workload with UI and prediction API
```
