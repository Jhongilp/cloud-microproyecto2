# cloud-microproyecto2
Archivos de configuración Azure Kubernetes Service

# Integrantes

- Jhos Hurtado
- Jonathan Gil



## Instalar Azure CLI using Docker

<aside>
💡 `docker run -it --rm -v ${PWD}:/work -w /work --entrypoint /bin/sh mcr.microsoft.com/azure-cli:2.42.0`

</aside>

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/67e2a5a3-2957-4842-8dc1-ffeeb678ca9f/Untitled.png)

## Login to az

<aside>
💡 az login

</aside>

## Create Resource Group

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d3ec4922-0c3d-42e7-892f-6a20d1fc9675/Untitled.png)

<aside>
💡 az group create -l westus -n `$RESOURCEGROUP`

</aside>

## Create Service Principal

Kubernetes needs a service account to manage our Kubernetes cluster

<aside>
💡 `SERVICE_PRINCIPAL_JSON=$(az ad sp create-for-rbac --skip-assignment --name aks-getting-started-sp -o json)`

</aside>

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/244cfb91-a250-40cd-a985-7714ecc68e7b/Untitled.png)

```markdown
{ 
	"appId": "650ee4ea-be8a-476d-be03-1012e8611f86", 
	"displayName": "aks-getting-started-sp", 
	"password": "x-L8Q~rSeVkaZdpNhebWMVpOtKXPLetZodDxia5p", 
	"tenant": "693cbea0-4ef9-4254-8977-76e05cb5f556" 
}
```

### Store variables locally

```markdown
# view and select your subscription account

az account list -o table
SUBSCRIPTION=<id>
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1128e27a-398d-4c4a-ad8d-e430a69d9cab/Untitled.png)

```markdown
SERVICE_PRINCIPAL=$(echo $SERVICE_PRINCIPAL_JSON | jq -r '.appId')
SERVICE_PRINCIPAL_SECRET=$(echo $SERVICE_PRINCIPAL_JSON | jq -r '.password')
```

### G**rant contributor role over the resource group to our service principal**

```markdown
az role assignment create --assignee $SERVICE_PRINCIPAL \
--scope "/subscriptions/$SUBSCRIPTION/resourceGroups/$RESOURCEGROUP" \
--role Contributor
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b70d12ef-bb67-4057-9db8-7c3e60b3cf74/Untitled.png)

### Create ssh key

```markdown
ssh-keygen -t rsa -b 4096 -N "aks_cloud_jgp" -C "jonathan.gil_pineda@uao.edu.co" -q -f  ~/.ssh/id_rsa
cp ~/.ssh/id_rsa* .
```

# Create the Cluster

```markdown
az aks create -n aks-getting-started \
--resource-group $RESOURCEGROUP \
--location eastus \
--load-balancer-sku standard \
--nodepool-name default \
--node-count 2 \
--node-vm-size standard_a2m_v2  \
--node-osdisk-size 250 \
--ssh-key-value ./id_rsa.pub \
--network-plugin kubenet \
--service-principal $SERVICE_PRINCIPAL \
--client-secret "$SERVICE_PRINCIPAL_SECRET" \
--output none
```

<aside>
💡 az aks list -o table

</aside>

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e21f7105-6d0d-443b-b6bc-b6041b85a3d1/Untitled.png)

### Credentials

```markdown
az aks get-credentials -n aks-getting-started \
--resource-group $RESOURCEGROUP
```

# ****Get kubectl****

```markdown
az aks install-cli
```

## Create namespace

```markdown
kubectl create ns example-app
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/32b5d465-394f-4440-95f2-5e58a67f3614/Untitled.png)

## Deploy

```markdown
kubectl apply -n example-app -f deployments/deployment.yaml
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1454392c-894e-4aa0-b8fe-d1753796a5a9/Untitled.png)

### Configmap

```markdown
kubectl apply -n example-app -f configmaps/configmap.yaml
```

### Secret

```markdown
kubectl apply -n example-app -f secrets/secret.yaml
```

### Service

```markdown
kubectl apply -n example-app -f services/service.yaml
```

You can check ..

```markdown
kubectl -n example-app get pods
kubectl -n example-app get deploy
#services
kubectl -n example-app get svc
```

# Remember to delete

```markdown
az group delete -n $RESOURCEGROUP
az ad sp delete --id $SERVICE_PRINCIPAL
```
