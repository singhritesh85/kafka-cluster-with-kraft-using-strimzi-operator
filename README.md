# kafka-cluster-with-kraft-using-strimzi-operator
```
# Add and update the repository
helm repo add strimzi https://strimzi.io/charts/
helm repo update

# Install the operator
helm upgrade --install strimzi-kafka-operator strimzi/strimzi-kafka-operator --namespace kafka --create-namespace --version 1.0.0 --kube-context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster
```
