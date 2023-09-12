# Instructions to deploy AKS Cluster

*NOTE* - `It's recommended to create a new service principal and add required permissions to it in order to proceed with further steps`

### Create a Service Principal first which will be used to run Terraform

```
az login
az account set --subscription 5135fe87-f70d-43dc-a7d5-ed71c8db7cac
az ad sp create-for-rbac --name <Service Account Name> --role Contributor --scopes /subscriptions/5135fe87-f70d-43dc-a7d5-ed71c8db7cac
```

### Add following environment variables to run Terraform

Client ID will be `appId` and Client Secret will be `password` provided as an output of above command. Take the values from above and add below and create the environment variables  

```
export ARM_SUBSCRIPTION_ID="5135fe87-f70d-43dc-a7d5-ed71c8db7cac"
export ARM_TENANT_ID="6f938018-ad23-46d8-ba0b-9a42b6a62129"
export ARM_CLIENT_ID="appId"
export ARM_CLIENT_SECRET="password"
export TF_VAR_service_principal_client_id="appId"
export TF_VAR_service_principal_client_secret="password"
```

### Add Azure AD permissions to the Service Principal 

- `Go to Azure Active Directory --> App Registrations --> <Service Account>`
- `Go to API permissions and click Add a permission`
- `Under Microsoft APIs, choose Microsoft Graph`
- `Under Microsoft Graph, choose Application permissions`
- `Add following permissions - Directory.ReadWrite.All, Group.ReadWrite.All` 

### Run Terraform 

Run following commands one by one to execute terraform 

```
terraform init
terraform plan
terraform apply 
```

### Once AKS cluster is up and running, add Cluster Admin permissions to your user

- `Go to your AKS cluster created in resource group terraform-aks-dev and choose Access Control (IAM) option`
- `Add role assignment to your Azure AD user`
- `Add following roles - Azure Kubernetes Service Cluster Admin Role, Azure Kubernetes Service RBAC Cluster Admin`

### Connect to AKS Cluster

Run following commands

```
az account set --subscription 5135fe87-f70d-43dc-a7d5-ed71c8db7cac
az aks get-credentials --resource-group terraform-aks-dev --name terraform-aks-dev-cluster
kubectl get ns
```

### Deploy the demo application

- `After a successful connection to the AKS Cluster run kubectl apply -f azure-demo-app.yaml`
- `Check the deployment as`
    ```
    kubectl -n azure-app get pods
    NAME                                READY   STATUS              RESTARTS   AGE
    azure-vote-back-8fd8d8db4-n8x5f     1/1     Running             0          10s
    azure-vote-front-5698dd7765-zfdnw   0/1     ContainerCreating   0          8s
    ```

    ```
    kubectl -n azure-app get svc
    NAME               TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)        AGE
    azure-vote-back    ClusterIP      10.0.129.181   <none>         6379/TCP       10m
    azure-vote-front   LoadBalancer   10.0.46.143    20.253.44.36   80:30106/TCP   10m
    ```
- `To access demo application from browser, go to network resource group terraform-aks-dev-nrg and look out for a Public IP Address type resource starting with kubernetes-*`
- `Within the Public IP address kubernetes-*, go to configuration option and add a unique name under DNS name label (optional)`
- `Save the changes and access your application through your browser`