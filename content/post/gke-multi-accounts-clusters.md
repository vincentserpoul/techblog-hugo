+++
date = "2018-06-13T11:37:00+08:00"
title = "Using GKE with multiple accounts and clusters"
description = "Using GKE with multiple accounts and clusters"
tags = [ "kubernetes", "GKE", "gcloud" ,"googlecloud" ]
topics = [ "kubernetes", "GKE", "gcloud" ,"googlecloud" ]
slug = "gke-multi-accounts-clusters"
draft = true
+++

## Google cloud configs

### Download and install gcloud

Simply follow [this link](https://cloud.google.com/sdk/docs/quickstarts)

### Listing existing configurations

```bash
gcloud config configurations list

NAME     IS_ACTIVE  ACCOUNT                         PROJECT          DEFAULT_ZONE       DEFAULT_REGION
```

This should give you only one line, with a default account.

### Creating a configuration

```bash
gcloud config configurations create testconf
```

This will guide you through the configuration of a new configuration. It allows you to set one config per account, project, cluster...

### Activating a configuration

```bash
gcloud config configurations activate testconf
```

As simple as this to change your account/config.

## Kubernetes cluster contexts

On linux, all kubernetes configs are stored in ~/.kube/config.
Each config is used via a context.

### Listing contexts

```bash
kc config get-contexts
```

will give you the list of available contexts.

### Creating a context

```bash
config set-context testconf
```

You will probably also need to get your cluster credentials

```bash
gcloud container clusters get-credentials clustername
```

### Activating a context

```bash
config use-context testconf
```

## Switching between projects with gcloud configs and kubernetes contexts

Once all the necessary configs and contexts were created, to switch from a cluster to another:

```bash
gcloud config configurations activate testconf
kubectl config use-context testconf
```
