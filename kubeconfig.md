# Devoriales - Kubeconfig

This document is a guide to understand and use the kubeconfig file.

## Table of Contents

- [What is Kubeconfig?](#what-is-kubeconfig)
- [Where is Kubeconfig stored?](#where-is-kubeconfig-stored)
- [Structure of Kubeconfig](#structure-of-kubeconfig)
  - [Clusters](#clusters)
  - [Contexts](#contexts)
  - [Users](#users)
- [Connect to a specific cluster](#connect-to-a-specific-cluster)


## What is Kubeconfig?

When you insteract with Kubernetes clusters, you need to provide the necessary information to authenticate and authorize your access to the cluster. This information is stored in a file called kubeconfig.

If you wouldn't have a kubeconfig file, you would have to provide the necessary information every time you interact with the cluster. This would be very cumbersome and error-prone.

Example when not using kubeconfig:
```bash
kubectl --server=https://<server> --certificate-authority=<ca> --client-key=<key
```

This would be very cumbersome and error-prone.

## Where is Kubeconfig stored?

The usual location of the kubeconfig file is in the home directory of the user. The file is named `config` and is located in the `.kube` directory.

```bash
~/.kube/config
```

You could also specify the location of the kubeconfig file using the `KUBECONFIG` environment variable.

```bash
export KUBECONFIG=/path/to/kubeconfig
```

or you could specify the kubeconfig file using the `--kubeconfig` flag.

```bash
kubectl --kubeconfig=/path/to/kubeconfig
```

## Structure of Kubeconfig

The kubeconfig file is a YAML file that contains the following information:

- clusters
- contexts
- users

### Clusters

In the `clusters` section, you define the Kubernetes clusters you want to interact with. Each cluster has a name and a server.

Example:
```yaml
clusters:
- name: my-cluster
  cluster:
    server: https://<server>
    certificate-authority-data: <ca>
```

### Contexts

The Contexts is a glue between the cluster and the user. It defines the cluster and the user to use when interacting with the cluster.

Example:
```yaml
contexts:
- name: my-context
  context:
    cluster: my-cluster
    user: my-user
```

### Users

The `users` section defines the users that can interact with the cluster.
You could have multiple users defined in the kubeconfig file.

Example:
```yaml
users:
- name: my-user
  user:
    client-certificate-data: <cert>
    client-key-data: <key>
```

You could either have the path to the certificate and key files instead of the data.

Example:
```yaml
users:
- name: my-user
  user:
    client-certificate: /path/to/cert
    client-key: /path/to/key
```

## Connect to a specific cluster

To connect to a specific cluster, you need to specify the context you want to use.

```bash
kubectl config use-context my-context
```

You could also specify the namespace you want to use.

```bash
kubectl config set-context --current --namespace=my-namespace
```

Youc an set a default namespace for each context.

```yaml
contexts:
- name: my-context
  context:
    cluster: my-cluster
    user: my-user
    namespace: my-namespace
```

