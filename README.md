# Grafana-Deployment(helm)
Prometheus + Grafana Deployment on Kubernetes with Helm

## 1. Add Official Helm Repositories:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```
 Explanation:
Adds the official charts for Prometheus and Grafana to your Helm repositories. helm repo update fetches the latest index from both.


## 2. Get Default Helm Values (Optional – for review/customization)

   ```bash
helm show values prometheus-community/prometheus > prom.values.yaml
helm show values grafana/grafana > grafana.values.yaml

   ```


## 3. Deploy Prometheus (with local-path storage auto-bound)

```bash
helm upgrade --install prometheus prometheus-community/prometheus \
  -n monitoring \
  --create-namespace \
  --set alertmanager.persistentVolume.storageClass="longhorn" \
  --set alertmanager.persistentVolume.enabled=true \
  --set server.persistentVolume.storageClass="longhorn" \
  --set server.persistentVolume.enabled=true \
  --set pushgateway.persistentVolume.storageClass="longhorn" \
  --set pushgateway.persistentVolume.enabled=true

```
Explanation:

Installs Prometheus into the monitoring namespace.

Ensures all PVCs (Alertmanager, Server, Pushgateway) use the default local-path StorageClass.

Avoids PVC binding issues commonly found in local/bare-metal clusters.

## 4. Deploy Grafana (NodePort + Persistent Local Storage)

```bash
helm upgrade --install grafana grafana/grafana \
  -n monitoring \
  --create-namespace \
  --set persistence.enabled=true \
  --set persistence.storageClassName="longhorn" \
  --set service.type=NodePort \
  --set adminPassword="admin123"



```

Explanation:

Grafana will be accessible via a NodePort.

Storage is persistent using the same local-path provisioner.

Admin password is set to admin123 (change it in production).


nodeport:
```bash
kubectl get svc grafana -n monitoring
```


 5. Default Grafana Dashboards to Import
Dashboard ID	Description
15757	Cluster Monitoring
15758	Namespace Monitoring
15759, 1860	Node Monitoring
5225	Pod Level Monitoring
13646	PVC Usage Monitoring

1.Access Grafana at:
```bash
http://<NodeIP>:<NodePort>
```

## 6. Prometheus URL (for Grafana Data Source)
```bash
http://prometheus-server.monitoring.svc.cluster.local
```

Use this URL when adding Prometheus as a data source in Grafana (under Configuration > Data Sources).


## 7. Post-deployment Check Commands

```bash
kubectl get pods -n monitoring
kubectl get pvc -n monitoring
kubectl get svc -n monitoring


```


