# dockeragent
build a docker image that will run as self-hosted Azure pipeline agent on kubernetes

Using the environment file we can create a ConfigMap with the command kubectl create cm azure-agent-config --from-env-file=<YOUR-FILE>.env With a Kubernetes Deployment we can run as many copies of the agent as we need, injecting our ConfigMap values along with an AZ_AGENT_NAME into the pod environments.

https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops
