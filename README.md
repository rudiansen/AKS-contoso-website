# Azure Kubernetes Service (AKS)

Below is the step-by-step for creating Azure Kubernetes Service (AKS) and deploying a static web application on it.

1. Create some variables

   ```bash
   export RESOURCE_GROUP=rg-contoso-video
   export CLUSTER_NAME=aks-contoso-video
   export LOCATION=eastasia
   ```
2. Create a Resource Group

   ```bash
   az group create -n $RESOURCE_GROUP -l $LOCATION
   ```

3. Create an AKS cluster

   ```bash
   az aks create -g $RESOURCE_GROUP -n $CLUSTER_NAME \
      --node-count 2 \
      --enable-addons http_application_routing \
      --generate-ssh-keys \
      --node-vm-size Standard_B2s \
      --network-plugin azure  
   ```

   The new AKS cluster will be created using [Standard_B2s](https://docs.microsoft.com/en-us/azure/virtual-machines/sizes-b-series-burstable)-sized VMs and the nodes will be part of **System** node.

4. Create a new Node pool

   ```bash
   az aks nodepool add -g $RESOURCE_GROUP \
      --cluster-name $CLUSTER_NAME \
      --name userpool \
      --node-count 2 \
      --node-vm-size Standard_B2s
   ```

   The above command adds a new node pool (**User mode**) which can be used to host applications and workloads.

5. Connect `kubectl` to new created AKS cluster

   ```bash
   az aks get-credentials -n $CLUSTER_NAME -g $RESOURCE_GROUP
   ```

6. Deploy contoso-website application to AKS cluster

   ```bash
   kubectl apply -f ./deployment.yml
   kubectl get deploy contoso-website
   ```

7. Create a service for the application

   ```bash
   kubectl apply -f ./service.yml
   kubectl get service contoso-website
   ```

8. Get the FQDN of the application from the cluster

   ```bash
   az aks show -g $RESOURCE_GROUP -n $CLUSTER_NAME \
      -o tsv --query addonProfiles.httpApplicationRouting.config.HTTPApplicationRoutingZoneName
   ```

   Copy the FQDN into the host element in [ingress.yml](ingress.yml) file.

9. Deploy an Ingress to make the application accessible from the internet

   ```bash
   kubectl apply -f ./ingress.yml
   kubectl get ingress contoso-website
   ```



  