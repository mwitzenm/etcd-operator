```
### Create the etcd-test project ( if you change this name you will need to modify the role binding )
oc new-project etcd-test 

### Create the operator crd
oc create -f etcd-operator-crd.yaml

### Verify the CRD was successfully created.
oc get crd
oc get etcdcluster

### Create the service account
oc create -f etcd-operator-sa.yaml

### Verify the Service Account was successfully created:
oc get sa

### Create the role
oc create -f etcd-operator-role.yaml

### Verify the Role was successfully created:
oc get roles

### Create the rolebinding
oc create -f etcd-operator-rolebinding.yaml

### Verify the RoleBinding was successfully created:
oc get rolebindings

### Create the deployment
oc create -f etcd-operator-deployment.yaml

### Verify the Etcd Operator Deployment was successfully created:
oc get deploy

### Verify the Etcd Operator Deployment pods are running:
oc get pods

### Open a new terminal window to follow Etcd Operator logs in real-time:

export ETCD_OPERATOR_POD=$(oc get pods -l name=etcd-operator -o jsonpath='{.items[0].metadata.name}' -o jsonpath='{.items[0].metadata.name}')
oc logs $ETCD_OPERATOR_POD -f

### Observe the leader-election lease on the Etcd Operator Endpoint:
oc get endpoints etcd-operator -o yaml

### Create the CRD
oc create -f etcd-operator-cr.yaml

### Verify the cluster object was created:
oc get etcdclusters

### Watch the pods in the Etcd cluster get created:
oc get pods -l etcd_cluster=example-etcd-cluster -w

### Verify the cluster has been exposed via a ClusterIP service:
oc get services -l etcd_cluster=example-etcd-cluster

### Let's now create another pod and attempt to connect to the etcd cluster via etcdctl:
oc run etcdclient --image=busybox busybox --restart=Never -- /usr/bin/tail -f /dev/null

### Access the pod:
kubectl exec -it etcdclient /bin/sh

### Install the Etcd Client:
wget https://github.com/coreos/etcd/releases/download/v3.1.4/etcd-v3.1.4-linux-amd64.tar.gz
tar -xvf etcd-v3.1.4-linux-amd64.tar.gz
cp etcd-v3.1.4-linux-amd64/etcdctl .

### Set the etcd version and endpoint variables:
export ETCDCTL_API=3
export ETCDCTL_ENDPOINTS=example-etcd-cluster-client:2379

### Attempt to write a key/value into the Etcd cluster:
./etcdctl put operator sdk
./etcdctl get operator

### Exit out of the client pod:
exit

### Let's change the size of the Etcd example-etcd-cluster CR. The Etcd Operator pod will detect the CR spec.size change and modify the number of pods in the cluster:
oc patch etcdcluster example-etcd-cluster --type='json' -p '[{"op": "replace", "path": "/spec/size", "value":5}]'

### Let's change the version of our example-etcd-cluster CR. The etcd-operator pod will detect the CR spec.version change and create a new cluster with the newly specified image:
oc patch etcdcluster example-etcd-cluster --type='json' -p '[{"op": "replace", "path": "/spec/version", "value":3.2.13}]'

### In another session delete a pod from the cluster and watch recovery in real-time.
export EXAMPLE_ETCD_CLUSTER_POD=$(oc get pods -l app=etcd -o jsonpath='{.items[0].metadata.name}' -o jsonpath='{.items[0].metadata.name}')
oc delete pod $EXAMPLE_ETCD_CLUSTER_POD

### Delete your Etcd cluster:
kubectl delete etcdcluster example-etcd-cluster

### Delete the Etcd Operator:
kubectl delete deployment etcd-operator

### Delete the Etcd CRD:
oc delete crd etcdclusters.etcd.database.coreos.com

```
