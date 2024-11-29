# dockeragent
build a docker image that will run as self-hosted Azure pipeline agent on kubernetes

Using the environment file we can create a ConfigMap with the command 
```
kubectl create cm azure-agent-config --from-env-file=var.env 
```

With a Kubernetes Deployment we can run as many copies of the agent as we need, injecting our ConfigMap values along with an AZ_AGENT_NAME into the pod environments.

https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops

Or, create a secret with following command.
```
kubectl create secret generic azdevops \
  --from-literal=AZP_URL=https://dev.azure.com/yourOrg \
  --from-literal=AZP_TOKEN=YourPAT \
  --from-literal=AZP_POOL=NameOfYourPool
kubectl apply -f deploy-dockeragent.yaml 
```
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: azdevops-deployment
  labels:
    app: azdevops-agent
spec:
  replicas: 1 #here is the configuration for the actual agent always running
  selector:
    matchLabels:
      app: azdevops-agent
  template:
    metadata:
      labels:
        app: azdevops-agent
    spec:
      containers:
      - name: kubepodcreation
        image: ezlee/dockeragent:18
        env:
          - name: AZP_URL
            valueFrom:
              secretKeyRef:
                name: azdevops
                key: AZP_URL
          - name: AZP_TOKEN
            valueFrom:
              secretKeyRef:
                name: azdevops
                key: AZP_TOKEN
          - name: AZP_POOL
            valueFrom:
              secretKeyRef:
                name: azdevops
                key: AZP_POOL
        volumeMounts:
        - mountPath: /var/run/docker.sock
          name: docker-volume
      volumes:
      - name: docker-volume
        hostPath:
          path: /var/run/docker.sock
```

## Setup and Usage

### Prerequisites

- Docker
- Kubernetes
- Azure DevOps account

### Installation

1. Clone the repository:
   ```
   git clone https://github.com/yourusername/dockeragent.git
   cd dockeragent
   ```

2. Build the Docker image:
   ```
   docker build -t yourusername/dockeragent:latest .
   ```

3. Create a ConfigMap from the environment file:
   ```
   kubectl create cm azure-agent-config --from-env-file=var.env
   ```

4. Deploy the agent to Kubernetes:
   ```
   kubectl apply -f deploy-dockeragent.yaml
   ```

## Examples of Common Use Cases

### Running Multiple Agents

To run multiple agents, simply increase the number of replicas in the Kubernetes Deployment configuration:
```
spec:
  replicas: 3
```

### Using a Different Azure DevOps Pool

To use a different Azure DevOps pool, update the `AZP_POOL` value in the environment file (`var.env`) or the Kubernetes secret:
```
kubectl create secret generic azdevops \
  --from-literal=AZP_URL=https://dev.azure.com/yourOrg \
  --from-literal=AZP_TOKEN=YourPAT \
  --from-literal=AZP_POOL=NewPoolName
kubectl apply -f deploy-dockeragent.yaml
```
