# Troubleshooting

Troubleshooting notes for the Intelligent Fog Orchestration System.

## 1. MQTT Broker Not Reachable

Check that the broker container is running:

```bash
docker ps
```

Check broker logs:

```bash
docker logs mqtt-broker
```

Expected broker port:

```text
1883
```

For local Docker Desktop runs, services usually connect with:

```text
MQTT_HOST=host.docker.internal
MQTT_PORT=1883
```

For LAN deployment, use the broker machine LAN IP:

```text
MQTT_HOST=192.168.x.x
```

For Tailscale deployment, use the broker machine Tailscale IP:

```text
MQTT_HOST=100.x.x.x
```

## 2. Fog Controller Not Opening

Check that the controller container is running:

```bash
docker ps
```

Check controller logs:

```bash
docker logs fog-controller
```

Expected local dashboard URL:

```text
http://localhost:8001
```

Check that the port is mapped:

```bash
docker ps
```

Expected mapping:

```text
8001:8001
```

## 3. IoT Smart Node Not Appearing Online

Check node logs:

```bash
docker logs node-1
docker logs node-2
```

Confirm node identity is set correctly:

```text
NODE_ID=node-1
NODE_ID=node-2
```

Confirm MQTT configuration:

```text
MQTT_HOST=host.docker.internal
MQTT_PORT=1883
```

For distributed runs, replace `host.docker.internal` with the broker machine LAN or Tailscale IP.

## 4. Workloads Not Starting

Check that the IoT Smart Node has Docker socket access:

```text
-v /var/run/docker.sock:/var/run/docker.sock
```

The node needs this mount because it starts and stops workload containers on the host Docker runtime.

Check node logs:

```bash
docker logs node-1
docker logs node-2
```

Check running containers:

```bash
docker ps
```

Check all containers, including stopped ones:

```bash
docker ps -a
```

## 5. Desired State Not Applied

Expected desired state:

```text
node-1:
  - promatch
  - fluidlab

node-2:
  - integral-calculator
  - cnn-edge-classifier
```

Check that the node ID matches the desired-state target.

Example:

```text
NODE_ID=node-1
```

Only `node-1` should apply workloads assigned to `node-1`.

Only `node-2` should apply workloads assigned to `node-2`.

Check controller logs:

```bash
docker logs fog-controller
```

Check node logs:

```bash
docker logs node-1
docker logs node-2
```

## 6. Reconciliation Not Working

Reconciliation depends on:

```text
node connected to MQTT
valid desired state
Docker socket mounted
managed workload configuration available
Docker engine running
```

Check Docker engine:

```bash
docker ps
```

Check node runtime logs:

```bash
docker logs node-1
docker logs node-2
```

A simple reconciliation test is to stop a managed workload manually and confirm that the IoT Smart Node starts it again.

## 7. ProMatch Not Opening

Expected local URL:

```text
http://localhost:3000
```

Check containers:

```bash
docker ps
```

Check node logs:

```bash
docker logs node-1
```

If running on another machine, replace `localhost` with the IP address of the machine running the workload.

## 8. FluidLab CFD Not Opening

Expected local URL:

```text
http://localhost:8082
```

Image used by the project:

```text
irfanuruchi/fluid-lab-cfd:cpu
```

Check containers:

```bash
docker ps
```

Check node logs:

```bash
docker logs node-1
```

## 9. Integral Calculator Not Opening

Expected local URL:

```text
http://localhost:5050
```

Check containers:

```bash
docker ps
```

Check node logs:

```bash
docker logs node-2
```

If explanation generation is not working, confirm that `CEREBRAS_API_KEY` is passed to `node-2`.

```text
CEREBRAS_API_KEY=$CEREBRAS_API_KEY
```

The key is used by the Integral Calculator workload, not by Fog Intelligence LLM.

## 10. CNN Edge Classifier Not Opening

Expected local URL:

```text
http://localhost:8600/ui
```

Image used by the project:

```text
irfanuruchi/cnn-edge-classifier:latest
```

Check containers:

```bash
docker ps
```

Check node logs:

```bash
docker logs node-2
```

If the UI opens but prediction fails, check the CNN workload container logs.

## 11. Fog Intelligence LLM Summary Not Working

Check that the Fog Intelligence LLM container is running:

```bash
docker ps
```

Check logs:

```bash
docker logs fog-intelligence-llm
```

The service connects to local Ollama using:

```text
OLLAMA_BASE_URL
OLLAMA_MODEL
```

Example:

```text
OLLAMA_BASE_URL=http://host.docker.internal:11434
OLLAMA_MODEL=llama3.2
```

Check that Ollama is running on the host machine and that the selected model is available.

The Fog Controller connects to Fog Intelligence LLM using:

```text
FOG_INTELLIGENCE_URL=http://host.docker.internal:8001
```

## 12. LAN or Tailscale Node Cannot Connect

Confirm that the MQTT broker machine is reachable from the node machine.

For LAN:

```text
MQTT_HOST=192.168.x.x
```

For Tailscale:

```text
MQTT_HOST=100.x.x.x
```

Check that both machines are online and that the broker port is reachable:

```text
1883
```

Use the broker machine IP, not the node machine IP.

## 13. Docker Image Architecture Issue

Verify image platforms:

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

## 14. Clean Restart

Stop and remove project containers:

```bash
docker rm -f mqtt-broker fog-controller fog-intelligence-llm node-1 node-2
```

Check remaining containers:

```bash
docker ps -a
```

Start the system again using the project run checklist.
sdasd

