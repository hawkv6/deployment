### General
The `jalapeno-helm-values.yaml` is used with https://github.com/jalapeno-api-gateway/jalapeno-helm to deploy Jalapeno.
The `jagw-helm-values.yaml` is used with https://github.com/jalapeno-api-gateway/jagw-helm/tree/main to deploy the Jalapeno API Gateway.

### Deploy Jalapeno
The hawkv6 Jalapeno will be installed in the `hawkv6-jalapeno` namespace.
To deploy the cluster, follow these steps.
Clone the `jalapeno-helm` repository:
	```
	git clone git@github.com:jalapeno-api-gateway/jalapeno-helm.git
	```
Copy the content of the `jalapeno-values.yaml` file into the `values.yaml` file in the folder `jalapeno-helm`
	```
	cat deployment/jalapeno-values.yaml > jalapeno-helm/jalapeno-helm/values.yaml
	``` 
Create the namespace `hawkv6-jalapeno` 
Install Jalapeno with the command
	```
	helm install jalapeno ./jalapeno-helm -n hawkv6-jalapeno
	```
This command has to be executed in the `jalapeno-helm` folder.

### Deploy Jalapeno API Gateway
The hawkv6 Jalapeno will be installed in the `hawkv6-jagw` namespace.
To deploy the cluster, follow these steps.
Clone the `jagw-helm` repository:
	```
	git clone git@github.com:jalapeno-api-gateway/jagw-helm.git
	```
Copy the content of the `jagw-values.yaml` file into the `values.yaml` file in the folder `jagw`
	```
	cat deployment/jagw-values.yaml > jagw-helm/jagw/values.yaml
	``` 
Create the namespace `hawkv6-jagw` 
Install Jalapeno with the command
	```
    cd jagw-helm
    helm install jagw jagw -n hawkv6-jagw 
	```
This command has to be executed in the `jagw-helm` folder.