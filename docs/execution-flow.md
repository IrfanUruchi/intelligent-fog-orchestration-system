# Execution Flow

Runtime execution flow for the Intelligent Fog Orchestration System.

This document describes the startup order, controller-node communication, desired-state publishing, Docker workload deployment, telemetry reporting, reconciliation behavior, and operational summary path.

## 1. Runtime Startup Order

The runtime services are started in this order:

```text
MQTT Broker
  -> Fog Intelligence LLM
  -> Fog Controller
  -> IoT Smart Nodes
      -> Docker workload containers
```

The MQTT broker is started first because both the Fog Controller and the IoT Smart Nodes use it for communication.

The Fog Intelligence LLM service can be started before or after the controller, but it must be available when the controller requests an operational summary.

## 2. Main Orchestration Path

The main orchestration path is:

```text
Fog Controller
  -> MQTT Broker
      -> IoT Smart Nodes
          -> Docker workloads
```

The Fog Controller publishes desired workload state through MQTT. IoT Smart Nodes receive the desired state and apply it to their local Docker runtime.

## 3. Operational Summary Path

The operational summary path is separate from workload orchestration:

```text
Fog Controller
  -> Fog Intelligence LLM
      -> local Ollama
```

The Fog Intelligence LLM service generates controller-level operational summaries. It does not communicate directly with IoT Smart Nodes and does not deploy workloads.

## 4. Controller Startup

After startup, the Fog Controller connects to the MQTT broker and prepares the dashboard.

The controller is responsible for:

```text
storing desired state
publishing desired-state updates
receiving node telemetry
tracking node status
recording orchestration events
requesting operational summaries
```

The controller does not directly run workload containers on remote nodes. Container execution is handled by the IoT Smart Nodes.

## 5. IoT Smart Node Startup

Each IoT Smart Node starts with a fixed node identity.

Project nodes:

```text
node-1
node-2
```

Each node connects to MQTT using the configured broker host and port.

Example:

```text
MQTT_HOST=host.docker.internal
MQTT_PORT=1883
```

For LAN or Tailscale deployments, `MQTT_HOST` is replaced with the broker machine IP address.

## 6. Node Registration and Telemetry

After connecting to MQTT, each IoT Smart Node reports its presence and begins publishing telemetry.

Telemetry is used by the controller to show:

```text
node online/offline state
workload status
runtime events
node-side updates
```

The controller dashboard uses this information to show the current state of the distributed system.

## 7. Desired-State Configuration

The project uses this desired state:

```text
node-1:
  - promatch
  - fluidlab

node-2:
  - integral-calculator
  - cnn-edge-classifier
```

The desired state defines which workloads should run on each node.

## 8. Desired-State Publishing

The Fog Controller publishes the desired state through MQTT.

Each IoT Smart Node receives the desired-state message and checks whether it applies to its own `NODE_ID`.

Example:

```text
node-1 receives desired state
node-1 reads only the workload list assigned to node-1
node-1 ignores workload placement assigned to node-2
```

## 9. Docker Runtime Inspection

After receiving desired state, the IoT Smart Node inspects the local Docker runtime.

The node compares:

```text
expected managed workloads
actual running managed workloads
```

If the local runtime already matches the desired state, no workload change is needed.

If a required workload is missing or stopped, the node starts it.

If a managed workload should no longer run on that node, the node stops it according to the configured behavior.

## 10. Workload Deployment

Workloads are started as Docker containers by the IoT Smart Node.

Project workload placement:

```text
node-1:
  ProMatch
  FluidLab CFD

node-2:
  Integral Calculator
  CNN Edge Classifier
```

The IoT Smart Node uses the workload configuration to start each container with the correct image, port mapping, environment variables, and managed labels.

## 11. Reconciliation Behavior

Desired-state reconciliation is the main control-loop behavior of the system.

```text
read desired state
inspect actual Docker state
compare expected vs actual workloads
apply correction when needed
report result back to controller
```

Example:

```text
Desired state:
node-1 should run promatch and fluidlab

Actual state:
promatch is running
fluidlab is stopped

Reconciliation:
node-1 starts fluidlab again
```

This avoids relying only on one-time start commands. The node keeps checking whether its runtime state matches the controller-defined state.

## 12. Runtime Monitoring

The IoT Smart Node monitors managed workload containers during execution.

Runtime monitoring supports:

```text
workload status tracking
telemetry updates
event reporting
self-healing behavior through reconciliation
```

The controller displays the resulting status and events in the dashboard.

## 13. Workload Access

After the workloads are running, local endpoints are:

```text
Fog Controller:         http://localhost:8001
ProMatch:               http://localhost:3000
FluidLab CFD:           http://localhost:8082
Integral Calculator:    http://localhost:5050
CNN Edge Classifier:    http://localhost:8600/ui
```

For LAN or Tailscale deployment, replace `localhost` with the IP address of the machine running the workload.

## 14. Fog Intelligence LLM Flow

The Fog Controller sends operational context to the Fog Intelligence LLM service.

The service sends the request to local Ollama using:

```text
OLLAMA_BASE_URL
OLLAMA_MODEL
```

The generated result is returned to the Fog Controller and displayed as an operational summary.

This summary is used for controller-level interpretation of system state. It is separate from the Integral Calculator explanation feature.

## 15. Integral Calculator Explanation Flow

The Integral Calculator workload has its own explanation support.

```text
IoT Smart Node 2
  -> Integral Calculator workload
      -> Cerebras explanation support
```

`CEREBRAS_API_KEY` is passed to `node-2` because the Integral Calculator workload uses it separately.

The Fog Intelligence LLM service does not use `CEREBRAS_API_KEY`.

## 16. CNN Edge Inference Flow

The CNN Edge Classifier runs as a Docker workload on `node-2`.

Runtime flow:

```text
open CNN Edge Classifier UI
upload image
run MobileNetV2 inference
return top predictions and confidence scores
show inference result in UI
```

This workload demonstrates CNN/DCNN inference as an edge workload managed by the same fog orchestration system.

## 17. Presentation Flow

A clean project presentation flow is:

```text
1. Show system architecture.
2. Start or show MQTT broker.
3. Open Fog Controller dashboard.
4. Show node-1 and node-2 connected through MQTT.
5. Show desired-state configuration.
6. Show workload placement on each node.
7. Open ProMatch and FluidLab CFD.
8. Open Integral Calculator and CNN Edge Classifier.
9. Run CNN image prediction.
10. Show telemetry and orchestration events.
11. Show Fog Intelligence LLM operational summary.
12. Stop one managed workload.
13. Show desired-state reconciliation.
```

## 18. Core Runtime Idea

The core runtime idea is:

```text
controller defines desired state
nodes observe actual runtime state
nodes reconcile mismatches locally
controller receives telemetry and events
controller uses LLM support for operational summaries
```

This is the main intelligent behavior of the project: a distributed control loop for fog workload orchestration.
