# Example of Prometheus helm chart configuration to ingest metrics to AMP

This is an example on how to set up ingestion from a new Prometheus server in Kubernetes to your Amazon Prometheus Service workspace.

## Prerequisites

- Kubernetes 1.16+
- Helm 3+

Get Prometheus Helm chart Repo Info
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add kube-state-metrics https://kubernetes.github.io/kube-state-metrics
helm repo update
```
See [helm repo](https://helm.sh/docs/helm/helm_repo/) for command documentation.

## Install Prometheus server helm chart

To install prometheus server reporting metrics from your Kubernetes cluster to Amazon Prometheus Service run the following command:

<pre><code>IAM_PROXY_PROMETHEUS_ROLE_ARN=<i>my_prometheus_iam_proxy_role_arn</i>
WORKSPACE_ID=<i>my_workspace_id</i>
AWS_REGION=<i>my_workspace_region</i>

helm install <i>prometheus-chart-name</i> prometheus-community/prometheus -n <i>prometheus-namespace</i> \
-f <i>my_prometheus_values_yaml</i>
-f https://raw.githubusercontent.com/aws-observability/amp-k8s-config-examples/main/prometheus/amp_ingest_override_values.yaml \
--set serviceAccounts.server.annotations."eks\.amazonaws\.com/role-arn"="${IAM_PROXY_PROMETHEUS_ROLE_ARN}" \
--set server.sidecarContainers.aws-sigv4-proxy-sidecar.args="{--name,aps,--region,${AWS_REGION},--host,aps-workspaces.${AWS_REGION}.amazonaws.com,--port,:8005}" \
--set server.remoteWrite[0].url="http://localhost:8005/workspaces/${WORKSPACE_ID}/api/v1/remote_write"
</code></pre>

In this command, `my_prometheus_iam_proxy_role_arn` is the ARN of the `amp-iamproxy-ingest-role` that you created following [Set up IAM roles for service accounts doc](https://docs.aws.amazon.com/prometheus/latest/userguide/AMP-onboard-ingest-metrics-existing-Prometheus.html#AMP-onboard-existing-Prometheus-IRSA). (the role ARN should have the format of `arn:aws:iam::your account ID:role/amp-iamproxy-ingest-role`), `my_workspace_id` is your Amazon Prometheus Service workspace ID, `my_workspace_region` is AWS region where your workspace is located,
`prometheus-chart-name` is your Prometheus release name,
`prometheus-namespace` is Kubernetes namespace prometheus server is installed into and `my_prometheus_values_yaml` is your main configuration file for Prometheus server.

To see all configurable options with detailed comments, visit the chart's [values.yaml](https://github.com/prometheus-community/helm-charts/blob/main/charts/prometheus/values.yaml).

See also [helm install](https://helm.sh/docs/helm/helm_install/) for command documentation.
