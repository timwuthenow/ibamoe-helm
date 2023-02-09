# Using helm to deploy IBAMOE - a work in progress

Deploy a pure [ubi image](https://catalog.redhat.com/software/containers/ubi8/ubi/5c359854d70cc534b3a3784e) as a container with [Helm](https://helm.sh/).
With the [values.yaml](https://github.com/timwuthenow/ibamoe-helm/blob/main/charts/ibamoe-helm/values.yaml) in the Helm chart we can configure [`replica count`](https://github.com/timwuthenow/ibamoe-helm/blob/main/charts/ibamoe-helm/values.yaml#L4) of the pods. The deployed containers are only a basic ubi operating system.

```yaml
replicaCount: 2
```

### Step 1: Log on to OpenShift

```sh
oc login --token=YOUR_TOKEN --server=YOUR_MASTERNODE_SERVER
```

### Step 2: Create an OpenShift project

```sh
export PROJECT_NAME=ibamoe-helm
oc new-project $PROJECT_NAME
```

### Step 3: Clone the repository

```sh
git clone https://github.com/timwuthenow/ibamoe-helm.git
```

### Step 4: Navigate to the Helm chart directory

```sh
export CHART_NAME=ibamoe-helm
cd ./charts/$CHART_NAME
```

### Step 5: Verify the values.yaml content

```sh
cat values.yaml
```

* Example output:

```sh
# Default values for chart.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.
replicaCount: 2
image:
  repository: "registry.redhat.io/ibm-bamoe/bamoe-businesscentral-rhel8"
  tag: latest
```

### Step 6: Verify the Helm chart

```sh
helm dependency update .
helm install --dry-run --debug $CHART_NAME .
helm lint
```

### Step 7: (optional) Create a [Helm package](https://helm.sh/docs/helm/helm_package/)

```sh
helm package .
```

* Example output:

```sh
Successfully packaged chart and saved it to: /ibamoe-helm/charts/ibamoe-helm/ibamoe-helm-v0.0.1.tgz
```

### Step 8: Install helm chart

```sh
helm install $CHART_NAME .
```

### Step 9: Verify the running pods

```sh
oc get pods
POD=$(oc get -n $PROJECT_NAME pods | grep $CHART_NAME | head -n 1 | awk '{print $1;}')
oc exec -n $PROJECT_NAME $POD --container $CHART_NAME -- ls
```

* Example output:

```sh
NAME                                 READY   STATUS             RESTARTS         AGE
ubi-helm-pam-bc-6464bb89f9-g29rv   1/1     Running   1 (106s ago)   6m48s
ubi-helm-pam-bc-6464bb89f9-jmt2s   1/1     Running   1 (106s ago)   6m48s
bin
boot
dev
etc
home
...
```

### Step 10: Uninstall helm chart

```sh
helm uninstall $CHART_NAME
```
