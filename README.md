# Kubernetes Meetup - Crossplane Demo

## Overview

This repository contains a sample project for demonstrating how to use Crossplane with Azure to deploy some Azure resources. Crossplane is an open-source Kubernetes add-on that enables platform teams to assemble infrastructure from various cloud providers, like Azure, AWS, or GCP, using Kubernetes-style API resources.

In this demo, I will walk you through the process of setting up Crossplane with Azure, configuring the Azure Provider, creating a Composition and deploying it along side with a Composite Resource Definition (XRD) and requesting Azure resources via a Claim.

### Prerequisites

Before starting, ensure you have the following installed:

1. Kubernetes cluster (with kubectl configured)
2. Azure CLI installed
3. Access to an Azure tenant and a subscription

### Setup

1. Install Crossplane (via Helm):

```bash
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
```

```bash
helm install crossplane \
    crossplane-stable/crossplane \
    --namespace crossplane-system \
    --create-namespace
```

2. Log in to the Azure CLI: `az login`

Create a service principal with access to an Azure subscription and save the resulting credentials:

```bash
az ad sp create-for-rbac \
--sdk-auth \
--role Owner \
--scopes /subscriptions/PLACE_YOUR_SUBSCRIPTION_ID_HERE > ./creds.json
```

3. Create a Kubernetes secret from the Service Principale credentials, for your Providers to use as the authentication method against Azure:

```bash
kubectl create secret \
generic azure-provider-credentials \
-n crossplane-system \
--from-file=credentials=./creds.json
```

4. Apply the preconfigured ProviderConfig from `configuration/providerconfig.yaml`

```bash
kubectl apply -f ./configuration/providerconfig.yaml
```

5. Install the Azure Providers:

```bash
kubectl apply -f ./configuration/provider.yaml
```

6. Install the Composite Resource Definition (XRD) and Composition:

```bash
kubectl apply -f ./apis/definition.yaml
```

```bash
kubectl apply -f ./apis/composition.yaml
```

7. Deploy a claim in a seperate namespace:

```bash
kubectl create namespace meetup
```

```bash
kubectl apply -f ./claim/meetup.yaml
```

8. Extend the Composition and XRD to also deploy and Virtual Network + Subnet and have fun :)