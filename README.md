# kubeauth

Maintain a distrubuted list of clusters and how to authenticate to them.

---

## How does it work

Kubeauth uses configured `ClusterSource`s to discover `ClusterConnection`s. These `ClusterConnection`s are then translated into kubeconfig entries.

---

## Install

### Windows (Chocolately)

```sh
choco install kubeauth
```

### MacOS and Linux

```sh
brew install kubeauth
```

---

## Git

For users you want to authenticate to your clusters via the cloud providers, `Git` is the recommended setup.

### Quickstart (Git)

1. Create a remote git repository
2. Add some `ClusterConnection`s as files to the remote git repository

AWS Example:
```yaml
# eks-cluster.yaml
apiVersion: kubeauth.io/v1alpha1
kind: ClusterConnection
metadata:
  name: eks-cluster
spec:
  provider: aws # (azure, gcp)
  region: us-east1
  clusterRef:
    name: eks-cluster
```

GCP Example:
```yaml
# gke-cluster.yaml
apiVersion: kubeauth.io/v1alpha1
kind: ClusterConnection
metadata:
  name: gke-cluster
spec:
  provider: gcp # (azure, gcp)
  region: us-central1
  clusterRef:
    name: gke-cluster
```
3. Configure the git `ClusterSource`

```sh
kubeauth add source root --type git --repo <git.url> --path <optional.subpath>
```

4. Setup the kubeconfig entries

```sh
kubeauth setup connections
```

Your kubeconfig file should now be setup to authenticate to your clusters via the cloud provider's `authCommand`.

### More Info (Git)

`ClusterSource`s can also be configured via files.

```yaml
apiVersion: kubeauth.io/v1alpha1
kind: ClusterSource
metadata:
  name: gke-clusters-source
spec:
  type: git
  repo: https://github.com/<ORG>/<NAME> # or ssh
```

This can allow for a git repository to define other git repositories as `ClusterSource`s. Allowing for highly distrubuted setups if desired.

See <example-repo-that-I-haven't-created-yet> as an example.

---

## SQL

For service accounts or applications where static access tokens are applicable, `SQL` is the recommended setup.

### Quickstart (SQL)

1. Create a SQL database like PostgreSQL.
2. Setup the database table for storing `ClusterConnection`s

```sh
kubeauth setup database --connection-string <your.connection.string>
```

***This command will create a table in the database used for storing cluster connection details***

3. Add `ClusterConnection`s

```sh
kubeauth database add --connection-string <your.connection.string> --cluster-connection-file <path.to.cluster.connection.file(s)>
```

4. Configure the sql `ClusterSource`

```sh
kubeauth add source root --type sql --connection-string <your.connection.string>
```

5. Setup the kubeconfig entries

```sh
kubeauth setup connections
```

### More Info (SQL)

TODO

---

## Convert KubeConfig

If you are starting with an existing set of clusters the below command will output a set of `ClusterConnection`s based on your local kubeconfig.

```sh
kubeauth convert kubeconfig --path ~/kube/.config --output ./cluster_connections.yaml
```

***TODO - Decide if want to invest in this functionality or let users do it***

---

## Developer Details

You can also use the go kubeauth client to connect to clusters programatically from configured sources.

```go
var gitSource := kubeauth.GitSource{
	Repo : "your.git.url"
}

var sqlSource := kubeauth.SQLSource{
	ConnectionString : "your.connection.string"
}

var kubeauthClient := kubeauth.New([gitSource, sqlSource])
```
