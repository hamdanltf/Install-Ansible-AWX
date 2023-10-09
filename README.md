# Install AWX

**system used in this documentation:**

1. OS: Ubuntu Server 22.04
2. CPUs: 4 cores
3. Memory: 8 GB
4. Storage: 80GB

also tested on:
1. Debian 12 Bookworm
2. Ubuntu Server 20.04

**Installation:**

install required packages

```bash
sudo apt install tar vim curl git wget ansible iptables -y
```

install K3s

```bash
curl -sfL https://get.k3s.io | sh -
```

```bash
sudo kubectl version --short

*kalo pake non-root user bakal error. ubah permission seperti di bawah
```

change K3s permission

```bash
sudo chown hamdan:hamdan /etc/rancher/k3s/k3s.yaml
```

check kube

```bash
watch kubectl get nodes

NAME          STATUS   ROLES                  AGE    VERSION
ansible-kvm   Ready    control-plane,master   140d   v1.25.6+k3s1
```


install kustomize

```bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
```

Ubah path kustomize

```bash
which kustomize
```
*harusnya error / atau kosong

```bash
sudo mv kustomize /usr/local/bin/
```

```bash
which kustomize

```
*harusnya udah bisa

create kustomization.yaml

```bash
vim kustomization.yaml
```

```bash
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=1.2.0

# Set the image tags to match the git version from above
images:
  - name: quay.io/ansible/awx-operator
    newTag: 1.2.0

# Specify a custom namespace in which to install AWX
namespace: awx
```

run kustomize

```bash
kustomize build . | kubectl apply -f -
```

wait pods ready

```bash
watch kubectl get pods --namespace awx
```

create awx.yaml

```bash
vim awx.yaml
```

```bash
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
spec:
  service_type: nodeport
  nodeport_port: 30080
```

edit kustomize.yaml

```bash
vim kustomization.yaml
```

```bash
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=1.2.0
  - awx.yaml

# Set the image tags to match the git version from above
images:
  - name: quay.io/ansible/awx-operator
    newTag: 1.2.0

# Specify a custom namespace in which to install AWX
namespace: awx
```

re-run kustomize

```bash
kustomize build . | kubectl apply -f -
```

wait pods to ready

```bash
watch kubectl get pods -n awx

NAME                                               READY   STATUS    RESTARTS      AGE
awx-postgres-13-0                                  1/1     Running   2 (34m ago)   26d
awx-operator-controller-manager-7dcc68565c-vlvs8   2/2     Running   0             24m
awx-task-6bd4b775c9-bzktk                          4/4     Running   0             22m
awx-web-58648bc74-pk9n5                            3/3     Running   0             20m
```

to view progres, open new terminal

```bash
kubectl logs -f deployments/awx-operator-controller-manager -c awx-manager --namespace awx
```

open dashboard on port 30080

to get dashboard admin user

```bash
kubectl get secret awx-admin-password -o jsonpath="{.data.password}" --namespace awx | base64 --decode
```





# Upgrade
edit kustomization.yaml

```bash
vim kustomization.yaml
```

```bash
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=2.4.0
  - awx.yaml

# Set the image tags to match the git version from above
images:
  - name: quay.io/ansible/awx-operator
    newTag: 2.4.0

# Specify a custom namespace in which to install AWX
namespace: awx
```

re-run kustomize

```bash
kustomize build . | kubectl apply -f -
```

wait pods to ready

```bash
watch kubectl get pods -n awx

NAME                                               READY   STATUS    RESTARTS      AGE
awx-postgres-13-0                                  1/1     Running   2 (34m ago)   26d
awx-operator-controller-manager-7dcc68565c-vlvs8   2/2     Running   0             24m
awx-task-6bd4b775c9-bzktk                          4/4     Running   0             22m
awx-web-58648bc74-pk9n5                            3/3     Running   0             20m
```
*sometimes you need to delete the pods

to view progres, open new terminal

```bash
kubectl logs -f deployments/awx-operator-controller-manager -c awx-manager --namespace awx
```

open dashboard on port 30080