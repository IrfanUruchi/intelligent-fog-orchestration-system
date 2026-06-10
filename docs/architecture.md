# System Architecture

The Intelligent Fog Orchestration System is a distributed fog orchestration platform for IoT and edge workloads.

The system follows a controller-node model. The Fog Controller defines the expected workload state. IoT Smart Nodes connect through MQTT, report telemetry, manage Docker workloads locally, and reconcile their runtime state against the desired state.

## 1. Architecture Paths

Runtime Startup order:
```text
MQTT Broker
  -> Fog Intelligence LLM
  -> Fog Controller
  -> IoT Smart Nodes
      -> Docker workloads
```

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

The Fog Intelligence LLM service is used by the Fog Controller to generate operational summaries. It is not part of the MQTT node control path and does not deploy workloads.

## 2. Main Components

```text
Fog Controller
MQTT Broker
IoT Smart Node
Docker Engine
Fog Intelligence LLM
Local Ollama runtime
Workload containers
```

## 3. Fog Controller

The Fog Controller is the central orchestration interface.

Main responsibilities:

```text
store desired state
publish desired state through MQTT
receive telemetry
track node status
record orchestration events
display workload state
request operational summaries from Fog Intelligence LLM
```

The controller is responsible for deciding what should run on each fog node. It does not directly start containers on remote machines. Workload execution is handled by the IoT Smart Nodes on their local Docker runtimes.

## 4. MQTT Broker

The MQTT broker is the communication layer between the Fog Controller and the IoT Smart Nodes.

It is used for:

```text
desired-state messages
node telemetry
node status updates
orchestration events
```

The broker makes the controller-node communication independent from direct HTTP calls between machines. This allows nodes to run locally, on the same LAN, or across remote machines through Tailscale.

## 5. IoT Smart Node

The IoT Smart Node is the runtime agent installed on each node.

Main responsibilities:

```text
connect to MQTT
identify itself using NODE_ID
receive desired-state updates
inspect local Docker runtime
start managed workload containers
stop managed workload containers
publish telemetry
report workload state
reconcile mismatches
```

The IoT Smart Node requires access to the host Docker socket because it controls workload containers on the local Docker engine.

```text
/var/run/docker.sock
```

The node does not decide the global workload placement. It applies the desired state assigned to its own `NODE_ID`.

## 6. Docker Workload Layer

Workloads are deployed as Docker containers.

Project workloads:

```text
ProMatch
FluidLab CFD
Integral Calculator
CNN Edge Classifier
```

This design keeps the node runtime generic. The IoT Smart Node does not need to know the internal implementation of each workload. It only needs the workload configuration, container image, ports, and required environment variables.

## 7. Fog Intelligence LLM

The Fog Intelligence LLM service provides controller-level operational summaries.

Runtime path:

```text
Fog Controller
  -> Fog Intelligence LLM
      -> local Ollama
```

Configuration:

```text
OLLAMA_BASE_URL
OLLAMA_MODEL
```

The service does not use Cerebras and does not communicate directly with IoT Smart Nodes. It receives operational context from the Fog Controller and returns a summary.

## 8. Integral Calculator Explanation Path

The Integral Calculator has its own explanation pipeline.

Runtime detail:

```text
IoT Smart Node 2
  -> Integral Calculator workload
      -> Cerebras explanation support
```

`CEREBRAS_API_KEY` is passed to `node-2` because the Integral Calculator workload uses it separately. This is independent from the Fog Intelligence LLM service.

## 9. Desired-State Model

The project uses the following desired state:

```text
node-1:
  - promatch
  - fluidlab

node-2:
  - integral-calculator
  - cnn-edge-classifier
```

The Fog Controller publishes this configuration through MQTT. Each IoT Smart Node checks whether the desired state applies to its own node ID.

## 10. Reconciliation Model

The core runtime behavior is desired-state reconciliation.

```text
desired state
  compared with
actual Docker runtime state
  then
missing or stopped managed workloads are corrected
```

Example:

```text
Desired state:
node-1 should run promatch and fluidlab

Actual state:
promatch is running
fluidlab is stopped

Action:
node-1 starts fluidlab again
```

This makes the system more reliable than one-time command execution because nodes continue checking whether their runtime state matches the expected state.

## 11. Deployment Modes

The same architecture supports local, LAN, and Tailscale deployment.

Local Docker Desktop setup:

```text
MQTT_HOST=host.docker.internal
```

LAN setup:

```text
MQTT_HOST=192.168.x.x
```

Tailscale setup:

```text
MQTT_HOST=100.x.x.x
```

The MQTT broker host is configurable, so fog nodes can connect from different machines while remaining under the same controller.

## 12. Architecture Diagram

```text
+------------------------------------------------------+
|                Fog Controller Dashboard              |
|------------------------------------------------------|
| Desired State | Node Status | Event Log | AI Summary |
+-----------------------------+------------------------+
              |                              |
              | MQTT                         | HTTP
              |                              v
              |              +-------------------------------+
              |              |     Fog Intelligence LLM       |
              |              |-------------------------------|
              |              | Local Ollama Operational Logic |
              |              +-------------------------------+
              |
              v
+-----------------------------+
|         MQTT Broker         |
+-----------------------------+
              |
              |
      +-------+-------+
      |               |
      v               v
+---------------------------+      +---------------------------+
|     IoT Smart Node 1      |      |     IoT Smart Node 2      |
|---------------------------|      |---------------------------|
| Docker Engine             |      | Docker Engine             |
| Desired-State Loop        |      | Desired-State Loop        |
| Telemetry Publisher       |      | Telemetry Publisher       |
| Runtime Monitor           |      | Runtime Monitor           |
|---------------------------|      |---------------------------|
| ProMatch                  |      | Integral Calculator       |
| FluidLab CFD              |      | CNN Edge Classifier       |
+---------------------------+      +---------------------------+
```

## 13. Component Repositories

```text
Fog Controller:
https://github.com/IrfanUruchi/fog-controller

IoT Smart Node:
https://github.com/IrfanUruchi/iot-smart-node

Fog Intelligence LLM:
https://github.com/IrfanUruchi/fog-intelligence-llm

CNN Edge Classifier:
https://github.com/IrfanUruchi/cnn-edge-classifier
```

## 14. Docker Images

```text
irfanuruchi/fog-controller:latest
irfanuruchi/iot-smart-node:latest
irfanuruchi/fog-intelligence-llm:latest
irfanuruchi/promatch:latest
irfanuruchi/fluid-lab-cfd:cpu
irfanuruchi/integral-calculator:latest
irfanuruchi/cnn-edge-classifier:latest
```

The published images support:

```text
linux/amd64
linux/arm64
```

