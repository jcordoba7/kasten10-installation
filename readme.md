##Installing and Setting up Kasten 10 on an OKE cluster


#### Step 1. Configure OCI Block Volume as the default storage class:

$ kubectl patch storageclass oci -p '{"metadata": {"annotations": {"storageclass.beta.kubernetes.io/is-default-class":"false"}}}'
$ kubectl patch storageclass oci-bv -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'


#### Step 2. Configure Kubernetes External Snapshotter:

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/client/config/crd/snapshot.storage.k8s.io_volumesnapshotclasses.yaml

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/client/config/crd/snapshot.storage.k8s.io_volumesnapshotcontents.yaml

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/master/client/config/crd/snapshot.storage.k8s.io_volumesnapshots.yaml


#### Step 3. Define a Volume Snapshot Class that leverages the blockvolume.csi.oraclecloud.com CSI Driver:

* Create a yaml file called 'volume-snapshot-class.yaml', with the following content:

apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: oci-bv-csi-snapclass
  annotations:
    k10.kasten.io/is-snapshot-class: "true"
driver: blockvolume.csi.oraclecloud.com
deletionPolicy: Delete


#### Step 4. Apply the file above on the cluster

$ kubectl apply -f volume-snapshot-class.yaml


#### Step 5. Set Basic Authentication for K10:

$ htpasswd -c -B -b htpass.pass geoff Mytoughpassword
$ htpasswd -B -b htpass.pass heather  passworddifficult
$ kubectl create ns kasten-io
$ kubectl create secret generic -n kasten-io kastenusers --from-file=htpass.pass


#### Step 6. Deploy Kasten K10 via the helm command:

$ helm repo add kasten https://charts.kasten.io/
$ helm install k10 kasten/k10 --namespace=kasten-io --create-namespace --set externalGateway.create=true --set auth.basicAuth.enabled=true --set auth.basicAuth.secretName=kastenusers


#### Step 7. Find the external IP of the Kasten Dashboard by using kubectl to query the gateway-ext service in OKE:

$ kubectl get services gateway-ext -n kasten-io | awk {'print $1" " $4" "$5'} | column -t

=====
NAME        EXTERNAL IP   PORT(S)
gateway-ext <external_ip> 80:31934/TCP
=====


#### Step 8. Navigate with your browser to http://<external_ip>/k10/#/ and wait a few minutes.
