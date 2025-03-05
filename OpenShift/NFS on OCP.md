### Task-1  Accessing the Master Node

Since the master node doesn’t have a public IP, use `oc debug` to access the master node
```
oc debug node/ip-10-0-15-132.ca-central-1.compute.internal
```
Switch to the host environment
```
chroot /host
```
Verify you’re on the master node
```
hostnamectl
```
* Note: Ensure that TCP/UDP ports 2049 (NFS) and 111 (RPCBind) are enabled.
### Task-2  Setting Up the NFS Server

On the ***`master node`***, check for the installed NFS utilities.

```
rpm -q nfs-utils
```
Create NFS Directory
```
sudo mkdir /tmp/mydbdata
```
Set Directory Permissions
```
sudo chown nobodynobody /tmp/mydbdata/
```
```
sudo chmod 777 /tmp/mydbdata/
```

Configure NFS Exports
```
sudo vi /etc/exports
```
Add the given line, by pressing `INSERT`
```
/tmp/mydbdata *(rw,sync,no_root_squash)
```
save the file using `ESCAPE + :wq!`

Apply the export
```
sudo exportfs -rv
```
Start and Enable NFS Server
```
sudo systemctl start nfs-server
sudo systemctl enable nfs-server
sudo systemctl restart nfs-server
```
Verify NFS export
```
showmount -e 10.0.15.132
```
* Replace the IP address with the master node’s IP.
  
### Task-3: Creating PersistentVolume (PV), PersistentVolumeClaim (PVC), and Pod on the Jump Server

Create the PV YAML File

```
vi nfs-pv.yaml
```

Add the given content, by pressing `INSERT`

```yaml
apiVersion v1
kind PersistentVolume
metadata
  name nfs-pv
spec
  storageClassName nfs-storage
  accessModes
  - ReadWriteMany
  capacity
    storage 2Gi
  nfs
    path "/tmp/mydbdata"
    server "10.0.15.132" #Master Node IP
    readOnly false
```

save the file using `ESCAPE + :wq!`

Apply the yaml
```
oc apply -f nfs-pv.yaml
```
Verify the PV
```bash
oc get pv
```

Create the PVC YAML File

```
vi nfs-pvc.yaml
```

Add the given content, by pressing `INSERT`

```yaml
apiVersion v1
kind PersistentVolumeClaim
metadata
  name nfs-pvc
spec
  storageClassName nfs-storage
  accessModes
    - ReadWriteMany
  resources
    requests
      storage 2Gi
```
save the file using `ESCAPE + :wq!`

Apply the yaml
```
oc apply -f nfs-pvc.yaml
```
Verify the PVC
```bash
oc get pvc
```

Create the Pod YAML File

```
vi nfs-pod.yaml
```

Add the given content, by pressing `INSERT`

```yaml
apiVersion v1
kind Pod
metadata
  name nfs-pod
spec
  containers
    - name httpd-ctr
      image httpdlatest
      volumeMounts
        - mountPath "/app"
          name nfs-volume
  volumes
    - name nfs-volume
      persistentVolumeClaim
        claimName nfs-pvc
```
save the file using `ESCAPE + :wq!`

Apply the yaml
```
oc apply -f nfs-pod.yaml
```
Verify the running Pod
```bash
oc get pod
```
```bash
oc describe pod nfs-pod
```
### Task-4: Cleaning Up Resources

When you’re done, clean up the created resources:
```
oc delete -f nfs-pod.yaml
```
```
oc apply -f nfs-pvc.yaml
```
```
oc apply -f nfs-pv.yaml
```
