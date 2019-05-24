To deploy OpenStack with Ceph by  K8S the following URL https://mirantis.jira.com/wiki/spaces/OSCORE/pages/1516503976/Deploy+OpenStack+on+k8s#DeployOpenStackonk8s-AUTOMATICDEPLOYMENT
should be used. And add `kubernetes_compute_count: 3` to context.

default_context:
  kubernetes_compute_count: 3


add rook-master https://charts.rook.io/master
helm search rook-ceph
helm install --namespace rook-ceph rook-master/rook-ceph --version 1.0.0`

`Import secret from ceph-rook namespace`

kubectl get secret rook-ceph-admin-keyring -n rook-ceph --export -o yaml | sed -e 's/keyring:/key:/' | kubectl apply -n openstack -f-

`Import ceph.conf configmap as well`

kubectl cp rook-ceph/$(kubectl -n rook-ceph get pod -l "app=rook-ceph-operator" -o jsonpath='{.items[0].metadata.name}'):/etc/ceph/ceph.conf /tmp/ && sed -i 's/[a-z]1\+://g' /tmp/ceph.conf

`REMOVE client.admin section`

kubectl create configmap rook-ceph-config -n openstack --from-file=/tmp/ceph.conf

`Install glance and nove with their values files`

https://raw.githubusercontent.com/ohryhorov/ceph-rook/master/values/glance-ceph.yaml

https://raw.githubusercontent.com/ohryhorov/ceph-rook/master/values/nova-ceph.yaml

`For example`

helm upgrade --install glance ./glance --namespace=openstack --values=./glance/glance-ceph.yaml

#for Debug
#kubectl -n rook-ceph exec -it $(kubectl -n rook-ceph get pod -l "app=rook-ceph-tools" -o jsonpath='{.items[0].metadata.name}') bash
