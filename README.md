# KCDLima-2024-kepler

# Installing Prometheus

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm install prometheus prometheus-community/kube-prometheus-stack \
    --namespace monitoring \
    --create-namespace \
    --wait

# Installing Kepler
helm repo add kepler https://sustainable-computing-io.github.io/kepler-helm-chart
helm repo update

helm install kepler kepler/kepler \
    --namespace kepler \
    --create-namespace \
    --set serviceMonitor.enabled=true \
    --set serviceMonitor.labels.release=prometheus \
    --set canMount.usrSrc=false

Post Install
KPLR_POD=$(
    kubectl get pod \
        -l app.kubernetes.io/name=kepler \
        -o jsonpath="{.items[0].metadata.name}" \
        -n kepler
)
kubectl wait --for=condition=Ready pod $KPLR_POD --timeout=-1s -n kepler

wget https://raw.githubusercontent.com/sustainable-computing-io/kepler/main/grafana-dashboards/Kepler-Exporter.json

GF_POD=$(
    kubectl get pod \
        -n monitoring \
        -l app.kubernetes.io/name=grafana \
        -o jsonpath="{.items[0].metadata.name}"
)
kubectl cp Kepler-Exporter.json monitoring/$GF_POD:/tmp/dashboards/kepler_dashboard.json


port-forward access the panel
kubectl port-forward svc/prometheus-grafana -n monitoring 8001:80 --address 0.0.0.0

Get user and password to login
kubectl get secret prometheus-grafana -n monitoring -o jsonpath='{.data.admin-password}' | base64 --decode
kubectl get secret prometheus-grafana -n monitoring -o jsonpath='{.data.admin-user}' | base64 --decode
