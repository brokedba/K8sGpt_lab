## Common Commands
### 1. Analyze Cluster Issues
Run a deep analysis of your Kubernetes cluster for common issues:
```bash
k8sgpt analyze
```
2. Analyze Specific Namespaces
```bash
k8sgpt analyze --namespace <namespace-name>
```

3. Set Output Language
```bash
k8sgpt config set language <language-code>

- Example:
k8sgpt config set language en
```
4. Specify Output Format
Change the output format to either text or json:

```bash
k8sgpt analyze --output json
```
5. Include Specific Filters
Focus the analysis on specific resource types or categories, such as Pods, Deployments, or Services:
Example:
```bash
 k8sgpt analyze --filter Pod --filter Ingress --filters Service
```
# Ask AI to find suggestions
use `--explain`
```bash
k8sgpt analyze --explain --filter=Service --namespace=default

k8sgpt analyze --explain --filter=Pod --namespace=broken
```
7. Anonymize during explain
```bash
k8sgpt analyze --explain --filter=Service --output=json --anonymize
```
# AI backend
List configured backends
```bash
k8sgpt auth list
Default:
> openai
Active:
Unused:
> openai
> localai
> ollama
> azureopenai
> cohere
> amazonbedrock
> amazonsagemaker
> google
> huggingface
> noopai
> googlevertexai
> ibmwatsonxai
```

# integrations
```bash
k8sgpt integrations list
Active:
> trivy
Unused:
> keda
> kyverno
> prometheus
> aws
k8sgpt integrations activate trivy

k8sgpt filters list | grep integration
> VulnerabilityReport (integration)
> ConfigAuditReport (integration)
```
- Use integration
```
k8sgpt analyze --filter=[integration(s)]

k8sgpt analyze --filter=VulnerabilityReport -n broken
```
# **Ollama**
```
k8sgpt analyze -f Pod -b ollama -e
```
Anonymize 
```
k8sgpt analyze -f Pod -b ollama -e --anonymize
```
