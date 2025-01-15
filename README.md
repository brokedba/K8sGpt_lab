# K8sGpt_lab

k8sgpt is a tool for scanning your Kubernetes clusters, diagnosing, and triaging issues in simple English.
It has SRE experience codified into its analyzers and helps to pull out the most relevant information to enrich it with AI. Out of the box integration with OpenAI, Azure, Cohere, Amazon Bedrock, Google Gemini and local models.
# Prerequisites:
- Kubernetes cluster configured and running.
- kubectl installed and configured to access your cluster.
- K8sGPT installed on your system. If not, install it by following the official documentation.

# Installation 
- **RHEL**
```bash
 curl -LO https://github.com/k8sgpt-ai/k8sgpt/releases/download/v0.3.48/k8sgpt_amd64.rpm
sudo rpm -ivh -i k8sgpt_amd64.rpm
```
- **Debian**
``` 
curl -LO https://github.com/k8sgpt-ai/k8sgpt/releases/download/v0.3.48/k8sgpt_amd64.deb
sudo dpkg -i k8sgpt_amd64.deb
k8sgpt version
k8sgpt: 0.3.24 (eac9f07), built at: unknown
```


- **Docker**
  
You can find the latest container image for K8sGPT in the packages of the GitHub organisation: Link
A volume can then be mounted to the image through e.g. Docker Compose. Below is an example:
```yaml
version: '2'
services:
 k8sgpt:
   image: ghcr.io/k8sgpt-ai/k8sgpt:dev-202304011623
   volumes:
     -  /home/$(whoami)/.k8sgpt.yaml:/home/root/.k8sgpt.yaml
 ```    
**In-Cluster installation(Operator)**

Installing the K8sGPT Operator Helm Chart
K8sGPT can be installed as an Operator inside the cluster. For further information, see the [K8sGPT Operator documentation.](https://docs.k8sgpt.ai/getting-started/in-cluster-operator/)

1.Install the operator  
```bash
helm repo add k8sgpt https://charts.k8sgpt.ai/
helm repo update
helm install release k8sgpt/k8sgpt-operator -n k8sgpt-operator-system --create-namespace
```
2. Create secret:
```bash
kubectl create secret generic k8sgpt-sample-secret --from-literal=openai-api-key=$OPENAI_TOKEN -n k8sgpt-operator-system
```
3. Apply the K8sGPT configuration object: example openai
```YAML
kubectl apply -f - << EOF
apiVersion: core.k8sgpt.ai/v1alpha1
kind: K8sGPT
metadata:
  name: k8sgpt-sample
  namespace: k8sgpt-operator-system
spec:
  ai:
    enabled: true
    model: gpt-3.5-turbo
    backend: openai
    secret:
      name: k8sgpt-sample-secret
      key: openai-api-key
    # anonymized: false
    # language: english
  noCache: false
  repository: ghcr.io/k8sgpt-ai/k8sgpt
  version: v0.3.8
  #integrations:
  # trivy:
  #  enabled: true
  #  namespace: trivy-system
  # filters:
  #   - Ingress
  # sink:
  #   type: slack
  #   webhook: <webhook-url> # use the sink secret if you want to keep your webhook url private
  #   secret:
  #     name: slack-webhook
  #     key: url
  #extraOptions:
  #   backstage:
  #     enabled: true
EOF
```
Once the custom resource has been applied the K8sGPT-deployment will be installed and you will be able to see the Results objects of the analysis (if there are any issues in your cluster):
```bash
❯ kubectl get results -o json | jq .
```
- output
```Json
{
  "apiVersion": "v1",
  "items": [
    {
      "apiVersion": "core.k8sgpt.ai/v1alpha1",
      "kind": "Result",
      "spec": {
        "details": "The error message means that the service in Kubernetes doesn't have any associated endpoints, which should have
 been labeled with \"control-plane=controller-manager\". \n\n
To solve this issue, you need to add the \"control-plane=controller-manager\" label to the endpoint that matches the service.
Once the endpoint is labeled correctly,Kubernetes can associate it with the service,
and the error should be resolved."
}
```
# K8sGPT Configuration

# **OpenAI**
![image](https://github.com/user-attachments/assets/d8e566b0-d448-41fb-9a64-92f61aeb1a26)


# **Ollama**

Ollama can get up and running locally with large language models. It runs Llama 2, Code Llama, and other models.

To start the Ollama server, follow the instruction in [Ollama](https://github.com/ollama/ollama?tab=readme-ov-file#start-ollama).
ollama serve
It can also run as an docker image, follow the instruction in [Ollama BLog](https://ollama.com/blog/ollama-is-now-available-as-an-official-docker-image)
```
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```
**Authenticate K8sGPT with Ollama:**
```
k8sgpt auth add --backend ollama --model llama3.1:latest --baseurl http://localhost:11434 
```
Analyze with a Ollama backend:
```
k8sgpt analyze --explain --backend ollama
```
3. **local AI**

- K8sGPT will need to be able to access both your local LLM, as well as your Kubernetes clusters.
- Access to the Kubernetes cluster should be set up as above using the kubectl configuration.
- As we are using LocalAI on our local machine, we can set a backend configuration using the gpt-4 model using the following command:
```
k8sgpt auth add --backend localai --model gpt-4 --baseurl http://localhost:8080
```
To test everything is working properly, we can run the following: 
```
k8sgpt analyze --explain --backend localai
```

# Integrations
```
$ k8sgpt integrations activate trivy
2024/08/22 18:01:54 creating 1 resource(s)
2024/08/22 18:01:54 beginning wait for 12 resources with timeout of 1m0s
2024/08/22 18:01:54 Clearing REST mapper cache
2024/08/22 18:01:56 creating 1 resource(s)
2024/08/22 18:01:56 creating 21 resource(s)
2024/08/22 18:01:56 release installed successfully: trivy-operator-k8sgpt/trivy-operator-0.24.1
```
check filters
```
user@Orpheus:~$ k8sgpt filters list
Active:
> Ingress
> VulnerabilityReport (integration)
> ConfigAuditReport (integration)
```
Check Vulnerability with a filter
```ini
$ k8sgpt analyze -f VulnerabilityReport
AI Provider: AI not used; --explain not set

0: VulnerabilityReport kube-system/daemonset-node-termination-handler-node-termination-handler(DaemonSet/node-termination-handler)
- Error: critical Vulnerability found ID: CVE-2022-1996 (learn more at: https://avd.aquasec.com/nvd/cve-2022-1996)
- Error: critical Vulnerability found ID: CVE-2023-24538 (learn more at: https://avd.aquasec.com/nvd/cve-2023-24538)
- Error: critical Vulnerability found ID: CVE-2023-24540 (learn more at: https://avd.aquasec.com/nvd/cve-2023-24540)
- Error: critical Vulnerability found ID: CVE-2024-24790 (learn more at: https://avd.aquasec.com/nvd/cve-2024-24790)

1: VulnerabilityReport cluster-tools/statefulset-prometheus-alertmanager-alertmanager(StatefulSet/prometheus-alertmanager)
- Error: critical Vulnerability found ID: CVE-2023-24538 (learn more at: https://avd.aquasec.com/nvd/cve-2023-24538)
- Error: critical Vulnerability found ID: CVE-2023-24540 (learn more at: https://avd.aquasec.com/nvd/cve-2023-24540)
- Error: critical Vulnerability found ID: CVE-2024-24790 (learn more at: https://avd.aquasec.com/nvd/cve-2024-24790)
- Error: critical Vulnerability found ID: CVE-2023-24538 (learn more at: https://avd.aquasec.com/nvd/cve-2023-24538)
- Error: critical Vulnerability found ID: CVE-2023-24540 (learn more at: https://avd.aquasec.com/nvd/cve-2023-24540)
- Error: critical Vulnerability found ID: CVE-2024-24790 (learn more at: https://avd.aquase
```
