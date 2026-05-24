# kafka-cluster-with-kraft-using-strimzi-operator
```
# Add and update the repository
helm repo add strimzi https://strimzi.io/charts/
helm repo update

# Install the operator
helm upgrade --install strimzi-kafka-operator strimzi/strimzi-kafka-operator --namespace kafka --create-namespace --version 1.0.0 --kube-context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster
```

```
kubectl apply -f kafka-kraft.yaml -n kafka --context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster

kubectl get kafka -n kafka --context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster
kubectl get knp -n kafka --context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster
kubectl get pods -n kafka --context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster

kubectl apply -f kafka-sasl-mtls-user.yaml --context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster	
```

-------------------------------------------------------------------------------------------------------------------------
For SASL_SSL
-------------------------------------------------------------------------------------------------------------------------	
kubectl get secret kafka-with-kraft-cluster-ca-cert -n kafka --context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster -o jsonpath='{.data.ca\.crt}' | base64 --decode > ca.crt	
keytool -import -trustcacerts -alias strimzi-kafka -file ca.crt -keystore truststore.jks -storepass Dexter@123 -deststoretype jks -deststorepass Dexter@123 -noprompt
kubectl get secret sasl-user -n kafka -o jsonpath='{.data.password}' --context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster | base64 --decode

cat sasl-client.properties

bootstrap.servers=kafka-with-kraft-kafka-bootstrap.kafka.svc.cluster.local:9093
security.protocol=SASL_SSL

# Trust configuration
ssl.truststore.location=truststore.jks
ssl.truststore.password=Dexter@123

# SASL configuration
sasl.mechanism=SCRAM-SHA-512
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
    username="sasl-user" \
    password="ZjXpwfkxatLnWi2YmgydkODhEbhMb1sW";
	
-------------------------------------------------------------------------------------------------------------------------
For mtls
-------------------------------------------------------------------------------------------------------------------------
kubectl get secret mtls-user -n kafka --context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster -o jsonpath='{.data.user\.crt}' | base64 --decode > user.crt
kubectl get secret mtls-user -n kafka --context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster -o jsonpath='{.data.user\.key}' | base64 --decode > user.key
openssl pkcs12 -export -in user.crt -inkey user.key -name mtls-user -out user.p12 -password pass:Dexter@123
keytool -importkeystore -srckeystore user.p12 -srcstoretype pkcs12 -srcstorepass Dexter@123 -destkeystore keystore.jks -deststoretype jks -deststorepass Dexter@123

cat mtls-client.properties

bootstrap.servers=kafka-with-kraft-kafka-bootstrap.kafka.svc.cluster.local:9094
security.protocol=SSL

# Truststore (To verify the broker)
ssl.truststore.location=/path/to/truststore.jks
ssl.truststore.password=Dexter@123

# Keystore (To authenticate the client to the broker)
ssl.keystore.location=/path/to/keystore.jks
ssl.keystore.password=Dexter@123

--------------------------------------------------------------------------------------------------------------------------
Connect from Client
--------------------------------------------------------------------------------------------------------------------------
kubectl cp sasl-client.properties kafka-with-kraft-kafka-with-kraft-broker-0:/tmp/sasl-client.properties -n kafka --context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster
kubectl cp mtls-client.properties kafka-with-kraft-kafka-with-kraft-broker-0:/tmp/mtls-client.properties -n kafka --context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster
kubectl cp truststore.jks kafka-with-kraft-kafka-with-kraft-broker-0:/tmp/truststore.jks -n kafka --context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster
kubectl cp keystore.jks kafka-with-kraft-kafka-with-kraft-broker-0:/tmp/keystore.jks -n kafka --context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster


kubectl exec -it kafka-with-kraft-kafka-with-kraft-broker-0 -n kafka --context=gke_wise-trainer-244916_us-central1_multicloud-gke-cluster -- bash
/opt/kafka/bin/kafka-topics.sh --bootstrap-server kafka-with-kraft-kafka-bootstrap.kafka.svc.cluster.local:9093 --create --topic secure-test --command-config /tmp/sasl-client.properties
/opt/kafka/bin/kafka-topics.sh --bootstrap-server kafka-with-kraft-kafka-bootstrap.kafka.svc.cluster.local:9094 --create --topic secure-test2 --command-config /tmp/mtls-client.properties
/opt/kafka/bin/kafka-topics.sh --bootstrap-server kafka-with-kraft-kafka-bootstrap.kafka.svc.cluster.local:9092 --create --topic secure-test3
