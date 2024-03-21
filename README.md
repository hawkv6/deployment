### General
#### Jalapeno Helm
The `jalapeno-helm-values.yaml` is used with https://github.com/jalapeno-api-gateway/jalapeno-helm to deploy Jalapeno.

#### JAGW Helm
The `jagw-helm-values.yaml` is used with https://github.com/jalapeno-api-gateway/jagw-helm/tree/main to deploy the Jalapeno API Gateway.

### Deploy Jalapeno
The hawkv6 Jalapeno will be installed in the `hawkv6-jalapeno` namespace.
To deploy the cluster, follow these steps.
1. Clone the `jalapeno-helm` repository:
	```
	git clone git@github.com:jalapeno-api-gateway/jalapeno-helm.git
	```
2. Copy the content of the `jalapeno-values.yaml` file into the `values.yaml` file in the folder `jalapeno-helm`
	```
	cat deployment/jalapeno-values.yaml > jalapeno-helm/jalapeno-helm/values.yaml
	``` 
3. Copy the content of the `collector-telegraf-ingress-cm.yaml` file into the config map used for the Telegraf ingress service. This will ensure that the correct topics in Kafka will be used to transfer data to the [clab-telemetry-linker](https://github.com/hawkv6/clab-telemetry-linker)
	```
	cat  deployment/collector-telegraf-ingress-cm.yaml > jalapeno-helm/jalapeno-helm/templates/collectors/telegraf-ingress/collector-telegraf-ingress-cm.yaml
	```
3. Copy the content of the `processors-telegraf-egress-cm.yaml` file into the config map used for the Telegraf egress service. This will ensure that the data are forwarded to the InfluxDB:
	```
	cat  deployment/processors-telegraf-egress-cm.yaml > jalapeno-helm/jalapeno-helm/templates/processors/telegraf-egress/processors-telegraf-egress-cm.yaml
	```
5. Create the namespace `hawkv6-jalapeno` 
6. Install Jalapeno with the command
	```
	helm install jalapeno ./jalapeno-helm -n hawkv6-jalapeno
	```
	This command has to be executed in the `jalapeno-helm` folder.

### Deploy Jalapeno API Gateway
The hawkv6 Jalapeno will be installed in the `hawkv6-jagw` namespace.
To deploy the cluster, follow these steps.
1. Clone the `jagw-helm` repository:
	```
	git clone git@github.com:jalapeno-api-gateway/jagw-helm.git
	```
2. Copy the content of the `jagw-values.yaml` file into the `values.yaml` file in the folder `jagw`
	```
	cat deployment/jagw-values.yaml > jagw-helm/jagw/values.yaml
	``` 
3. Create the namespace `hawkv6-jagw` 
4. Install Jalapeno with the command
	```
    cd jagw-helm
    helm install jagw jagw -n hawkv6-jagw 
	``` 
	This command has to be executed in the `jagw-helm` folder.