# OpenShift Tasks
## Login

### 1. How to login to the cluser using oc

`oc login -u username -p passw0rd https://api.cluster-1563.1563.sandbox283.opentlc.com:6443`

### 2. How to su -

If you have enough permissions to impersonate, obviously. You only need to use the flag --as.

For example:

run the command as jorge user

`oc --as=jorge get pods`

Also, a group impersonation can be done, instead of a user impersonation:

`oc --as-group=developers get pods`

It’s very handy and quick in many situations, for example, to check if a user can perform a specific action or to check the output a user would receive when running oc. It’s also useful when messing with roles and permissions.

### 3. How to imposonate system:admin

`oc --as=system:admin .....`

### 4. How access cluster using TLS certificate (X.590)***
`export KUBECONFIG=/home/user/auth/kubeconfig` </p>
`oc <command>`

or

`oc <command> --kubeconfig /home/user/auth/kubeconfig`


### 5. How to login using token.

`oc login --token <token>`

## Users / Groups

### 1. how to add user accounts from htpasswd

The HTPasswd identity provider validates users against a secret that that contains user names and passwords generated with the htpasswd command from the Apache HTTP Server project. Only a cluster administrator is allowed to change the data inside the secret. Regular users cannot change their own passwords.


1.1 create htpasswd file </p>

`htpasswd -c -B -b </path/to/users.htpasswd> <user_name> <password> `. </p>

Example </p>

`htpasswd -c -B -b users.htpasswd user1 MyPassword!`

continue to add user </p>

`htpasswd -B -b </path/to/users.htpasswd> <user_name> <password>`

delete user </p>

`htpasswd -D /tmp/htpasswd student`

1.2 create htpasswd file secret in openshift-config project.

`oc create secret generic htpass-secret --from-file=htpasswd=</path/to/users.htpasswd> -n openshift-config`

1.3 create CRD for htpasswd

```
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: my_htpasswd_provider 
    mappingMethod: claim 
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpasswd
```

`oc apply -f </path/to/CR>`

* user needs to login to the cluster before it appears in `oc get users`

### 2. how to update user in htpasswd

Add more user </p>

`htpasswd -B -b </path/to/users.htpasswd> <user_name> <password>`

update htpasswd secret

`oc create secret generic htpasswd --from-file=htpasswd=htpasswd --dry-run -o yaml -n openshift-config | oc replace -f -`

### 3. how to delete htpasswd user

if you delete user from identity provider, the user and identity resources must be removed as well.

delete user </p>

`htpasswd -D /tmp/htpasswd student`

update htpasswd secret

`oc create secret generic htpass-secret --from-file=htpasswd=users.htpasswd --dry-run -o yaml -n openshift-config | oc replace -f -`

```
oc delete user <username>
oc get identities | grep <username>
oc delete identity my_htpasswd_provider:<username>
oc delete identity <username>
```

### 4. How to extract secret data
`
 oc extract secret/htpasswd -n openshift-config --to - > /tmp/htpasswd`

### 2. How to remove kubeadmin user

`oc delete secret kubeadmin -n kube-system`

### 3. How to create group

oc adm groups new dev-group
oc adm groups add-users dev-group developer

### 7. How to create service accounts.

`oc create sa <service_account_name> `

## RBAC

Default cluster roles


**Admin**
A project manager. If used in a local binding, an admin has rights to view any resource in the project and modify any resource in the project except for quota.

**basic-user**
A user that can get basic information about projects and users.

**cluster-admin** A super-user that can perform any action in any project. When bound to a user with a local binding, they have full control over quota and every action on every resource in the project.

**cluster-status** A user that can get basic cluster status information.

**edit** A user that can modify most objects in a project but does not have the power to view or modify roles or bindings.

**self-provisioner** A user that can create their own projects.

**view** A user who cannot make any modifications, but can see most objects in a project. They cannot view or modify roles or bindings.

### 1. how to assign cluster administrative privileges (cluster-admin) to user.
```
$ oc adm policy add-cluster-role-to-user cluster-admin keith
clusterrole.rbac.authorization.k8s.io/cluster-admin added: "keith"

$ oc get clusterrolebindings cluster-admin-0 -o yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  creationTimestamp: "2020-08-21T03:17:09Z"
  name: cluster-admin-0
  resourceVersion: "590469"
  selfLink: /apis/rbac.authorization.k8s.io/v1/clusterrolebindings/cluster-admin-0
  uid: 66ad0ced-b094-4eee-8ca0-3bfccf123a2b
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: keith
  
```

### 2. how to create developer user (edit role)

oc policy add-role-to-group edit dev-group
oc policy add-role-to-group edit <username> -n project-name (optional)

### 3. How to assign sudoer role to user.

`oc create clusterrolebinding <any_valid_name> --clusterrole=sudoer --user=<username>`

Example:

```
$ oc create clusterrolebinding bank-as-sudoer --clusterrole=sudoer --user=bank
clusterrolebinding.rbac.authorization.k8s.io/bank-as-sudoer created

$ oc get clusterrolebinding.rbac.authorization.k8s.io/bank-as-sudoer -o yaml

$ oc --as=system:admin whoami
system:admin

$ oc get secret -n openshift-config
Error from server (Forbidden): secrets is forbidden: User "bank" cannot list resource "secrets" in API group "" in the namespace "openshift-config"

$ oc --as=system:admin get secret -n openshift-config
NAME                                  TYPE                                  DATA   AGE
builder-dockercfg-2df5d               kubernetes.io/dockercfg               1      3d14h
builder-token-2g5pk                   kubernetes.io/service-account-token   4      3d14h
builder-token-8p2p7                   kubernetes.io/service-account-token   4      3d14h
default-dockercfg-nbg7w               kubernetes.io/dockercfg               1      3d14h
default-token-drwtv                   kubernetes.io/service-account-token   4      3d14h
default-token-jfx72                   kubernetes.io/service-account-token   4      3d14h
deployer-dockercfg-km5n4              kubernetes.io/dockercfg               1      3d14h
deployer-token-pnx8s                  kubernetes.io/service-account-token   4      3d14h
deployer-token-vm7p7                  kubernetes.io/service-account-token   4      3d14h
etcd-client                           SecretTypeTLS                         2      3d14h
etcd-metric-client                    SecretTypeTLS                         2      3d14h
etcd-metric-signer                    SecretTypeTLS                         2      3d14h
etcd-signer                           SecretTypeTLS                         2      3d14h
htpasswd                              Opaque                                1      3d10h
initial-service-account-private-key   Opaque                                1      3d14h
pull-secret                           kubernetes.io/dockerconfigjson        1      3d14h

```
### 4. How to remove self-provisioner role from authenticated user:

 oc adm policy remove-cluster-role-from-group \ > self-provisioner system:authenticated:oauth
 
 
### 5. how to assign project admin privilege to user
oc adm policy add-role-to-user admin keith -n project-name

### 6. how to assign read-only permission to user
oc adm policy add-role-to-user view keith -n project-name

### 7. how to assign full control over the project including rate limit

oc adm policy add-cluster-role-to-user cluster-admin keith


### 8. how to test if user can perform tasks.

oc adm policy who-can delete user

## Projects


### 1. how to create openshift project

oc new-project hello-openshift \
    --description="This is an example project" \
    --display-name="Hello OpenShift"
    
### 2. How to switch between project.

oc project <projectname>

### 3. How to delete project
oc delete project <project_name>

### 4. how to allow other user to work on my project


## Build
### 1. How to create build config
### 2. How to start a new build

## Deploy
### 1. How to deploy new application from Source code
### 2. How to deploy new application from Dockerfile
### 3. HOw to deploy new application from Docker Image

## Image Stream & Image Stream Tag

## Resource Management
1. limit resource usage

## Upgrade
1. How to upgrade OpenShift Cluster
## Scale
1. scale application to meet increased demand

## Event & Alert & logs

1. HOw to monitor cluster status
2. How to monitor cluster event and alerts
3. How to view logs 
4. How to view log of failed/terminated pods.
5. 
## Kubernetes Resources
1. How to import Kubernetes resources


## Networking - Routes, Services, Ingress
1. How to create route
2. How to remove route
3. Create self signed certificate
4. Secure route using TLS certificate

## Rollout

## Cluster Scaling.

## Image Registry

## Installation

## Troubleshooting 
### 0. get cluster version
> oc get clusterversion </p>
> NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS </p>
> version   4.4.8     True        False         3d13h   Cluster version is 4.4.8

`oc describe clusterversion`

### review cluster operators
`oc get clusteroperators`


### 1. debug pod
`oc logs pod/podname` </p>
`oc debug pod/podname`

### 2. debug node
`oc get nodes` </p>
`oc adm top nodes` </p>
`oc describe node my-node-name` </p>
`oc adm node-logs -u crio my-node-name` </p>
`oc adm node-logs -u kubelet my-node-name` </p>
`oc adm node-logs my-node-name` </p>

open shell prompt to node:

`oc debug node/my-node-name`

> $ oc debug node/ip-10-0-138-229.ap-southeast-1.compute.internal
> 
> Starting pod/ip-10-0-138-229ap-southeast-1computeinternal-debug ...
> To use host binaries, run `chroot /host`
> 
> chroot /host
> Pod IP: 10.0.138.229
> If you don't see a command prompt, try pressing enter.
> 
> sh-4.2# chroot /host </p>
> sh-4.4# cat /etc/*release*
> 
> NAME="Red Hat Enterprise Linux CoreOS"
> 
> VERSION="44.81.202006080130-0"
> 
> VERSION_ID="4.4"
> 
> OPENSHIFT_VERSION="4.4"
> 
> RHEL_VERSION="8.2"
> 
> PRETTY_NAME="Red Hat Enterprise Linux CoreOS 44.81.202006080130-0 (Ootpa)"
> 
> ID="rhcos"
> 

list container on nodes </P>
`crictl ps`


> $ oc adm top node
> NAME                                              CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
> ip-10-0-138-229.ap-southeast-1.compute.internal   330m         4%     2541Mi          8%
> ip-10-0-152-138.ap-southeast-1.compute.internal   509m         14%    3258Mi          22%
> ip-10-0-179-0.ap-southeast-1.compute.internal     421m         5%     3373Mi          11%
> ip-10-0-189-168.ap-southeast-1.compute.internal   508m         14%    3221Mi          22%
> ip-10-0-214-156.ap-southeast-1.compute.internal   626m         17%    3606Mi          24%
> thanachit@BankMBP ~/Projects/openshift-task/users (master)

> $ oc adm top pod
> NAME                      CPU(cores)   MEMORY(bytes)
> example-75778c488-9nz95   0m           1Mi
> example-75778c488-h7n9g   0m           1Mi
> example-75778c488-pd6rw   0m           2Mi
> my-simple-web-9-68xr7     0m           28Mi
> my-simple-web-9-7n6bp     0m           20Mi


### 3. oc with debug

`oc --loglevel 7 get pod`

## Persistent Storage 

