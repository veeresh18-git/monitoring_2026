# monitoring_2026


# ==========================================
# PROMETHEUS + GRAFANA MINIKUBE HANDS-ON LAB
# ==========================================

# STEP 1: CHECK PREREQUISITES

docker --version
minikube version
kubectl version --client
helm version


# STEP 2: START MINIKUBE

minikube start --driver=docker --cpus=4 --memory=8192

# Check Minikube status
minikube status

# Check Kubernetes nodes
kubectl get nodes

# Check system pods
kubectl get pods -n kube-system


# STEP 3: ADD PROMETHEUS HELM REPOSITORY

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

# Update Helm repositories
helm repo update

# Check available charts
helm search repo prometheus-community/kube-prometheus-stack


# STEP 4: INSTALL PROMETHEUS + GRAFANA

helm install monitoring prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace

# Check namespace
kubectl get namespaces

# Check monitoring pods
kubectl get pods -n monitoring

# Watch pods until ready
kubectl get pods -n monitoring -w

# Check services
kubectl get svc -n monitoring

# Check Helm release
helm list -n monitoring


# STEP 5: ACCESS PROMETHEUS
# Run this in a separate PowerShell terminal.
# Keep this terminal open.

kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090:9090 -n monitoring

# Open in browser:
# http://localhost:9090
# http://localhost:9090/targets


# STEP 6: PROMETHEUS QUERIES
# Enter these in Prometheus UI -> Query

up

kube_pod_info

node_cpu_seconds_total

node_memory_MemAvailable_bytes

kube_pod_container_status_restarts_total


# STEP 7: ACCESS GRAFANA
# Run this in another PowerShell terminal.
# Keep this terminal open.

kubectl port-forward svc/monitoring-grafana 3000:80 -n monitoring

# Open in browser:
# http://localhost:3000

# Username:
# admin

# Get Grafana password in PowerShell:

$encoded = kubectl get secret monitoring-grafana -n monitoring -o jsonpath="{.data.admin-password}"
[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($encoded))


# STEP 8: DEPLOY NGINX APPLICATION

kubectl create deployment nginx --image=nginx

# Check deployment
kubectl get deployments

# Check pods
kubectl get pods

# Scale Nginx to 3 replicas
kubectl scale deployment nginx --replicas=3

# Check pods again
kubectl get pods

# Check all resources
kubectl get all


# STEP 9: PROMQL QUERIES FOR NGINX PODS
# Enter these in Grafana -> Explore -> Prometheus

# Check target availability
up

# Check Kubernetes pod information
kube_pod_info

# Check container memory usage
container_memory_working_set_bytes{namespace="default",container!="",container!="POD"}

# Check CPU usage per pod (cores)
sum by (pod) (
  rate(container_cpu_usage_seconds_total{namespace="default",container!="",container!="POD"}[5m])
)

# Check container restarts
kube_pod_container_status_restarts_total{namespace="default"}


# STEP 10: SET CPU AND MEMORY REQUESTS/LIMITS

kubectl set resources deployment nginx --requests=cpu=100m,memory=64Mi --limits=cpu=200m,memory=128Mi

# Verify resources
kubectl describe deployment nginx


# STEP 11: CHECK PODS AND NODES

kubectl get nodes
kubectl get pods -A
kubectl get deployments -A
kubectl get svc -A

# Check monitoring logs
kubectl logs -l app.kubernetes.io/name=prometheus -n monitoring --tail=50

# Check Grafana logs
kubectl logs -l app.kubernetes.io/name=grafana -n monitoring --tail=50


# STEP 12: TROUBLESHOOTING

# Describe a pod
kubectl describe pod POD_NAME -n monitoring

# View pod logs
kubectl logs POD_NAME -n monitoring

# View events
kubectl get events -n monitoring --sort-by=.metadata.creationTimestamp

# Check Minikube resources
kubectl describe node minikube

# Check all monitoring resources
kubectl get all -n monitoring



kube_deployment_spec_replicas{deployment="nginx-demo"}
kube_deployment_status_replicas_available{deployment="nginx-demo"}
count(kube_pod_status_phase{namespace="default",pod=~"nginx-demo-.*",phase="Running"} == 1)


# STEP 13: CLEANUP

# Remove Nginx deployment
kubectl delete deployment nginx

# Uninstall Helm release
helm uninstall monitoring -n monitoring

# Delete monitoring namespace
kubectl delete namespace monitoring

# Stop Minikube
minikube stop

# ==========================================
# END OF LAB
# ==========================================
