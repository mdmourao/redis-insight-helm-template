# Redis Insight Helm Chart

This Helm chart is an adaptation of the official [Redis Insight deployment instructions](https://redis.io/docs/latest/operate/redisinsight/install/install-on-k8s/) converted to a Helm template format.

## Usage

Clone this repository and customize the `values.yaml` file according to your needs:

```bash
git clone <repository-url>
cd redis-insight-helm-template
# Edit values.yaml as needed
helm install redisinsight .
```

Start porting forwarding to access Redis Insight:

```bash
kubectl port-forward svc/redisinsight-service 5540:80 -n default
```

And then open your browser and navigate to `http://localhost:5540`.

### Uninstalling

To uninstall the Redis Insight deployment, run:

```bash
helm uninstall redisinsight
```

## Recommended Configuration Changes

It is recommended to change the following values in `values.yaml` before deploying:

```yaml
namespace: monitor

image:
  tag: 3.0.2

persistence:
  storageClassName: standard # eg: for GKE

resources:
  requests:
    memory: "256Mi"
    cpu: "300m"
  limits:
    memory: "1Gi"
    cpu: "500m"
```

* `namespace`: Change to your desired Kubernetes namespace.
* `image.tag`: Update to a locked version of Redis Insight for stability.
* `resources`: Adjust resource requests and limits based on your cluster capacity and expected load. Not on official recommendations, but useful for production environments.
* `persistence.storageClassName`: Set to your cluster's storage class for persistent volume claims.

## Changes from Official Documentation

The only change from the official Redis documentation is the **Service type**:

| Setting | Official Docs | This Chart |
|---------|---------------|------------|
| Service Type | `LoadBalancer` | `ClusterIP` |

This change was made for **default security reasons** — using `ClusterIP` prevents the service from being directly exposed to the internet. You can access RedisInsight using `kubectl port-forward` or by setting up an Ingress controller.

If you need a LoadBalancer, simply update `values.yaml`:

```yaml
service:
  type: LoadBalancer
```

## Resources Created

This Helm chart creates the following Kubernetes resources:

| Resource | Name | Description |
|----------|------|-------------|
| **Deployment** | `<release-name>` | Runs the RedisInsight container with an init container for proper file permissions |
| **Service** | `<release-name>-service` | Exposes RedisInsight on the configured port (default: ClusterIP) |
| **PersistentVolumeClaim** | `<release-name>-pv-claim` | Provides persistent storage for RedisInsight data |
