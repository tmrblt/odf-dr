---
### Create htpasswd file with admin user
htpasswd -c -B -b users.htpasswd admin redhat

### Add or update users with existing file
htpasswd -B -b users.htpasswd user1 redhat
htpasswd -B -b users.htpasswd user2 redhat
htpasswd -B -b users.htpasswd user3 redhat
htpasswd -B -b users.htpasswd user4 redhat
htpasswd -B -b users.htpasswd user5 redhat

# create secret
oc create secret generic htpass-secret --from-file=htpasswd=users.htpasswd -n openshift-config

## create oauth cr
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: htpasswd
    mappingMethod: claim 
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret 

## admin role
 oc adm policy add-cluster-role-to-user <role> <username>

 oc adm policy add-cluster-role-to-user cluster-admin admin

### retrieve secret
oc get secret htpass-secret -ojsonpath={.data.htpasswd} -n openshift-config | base64 --decode > users.htpasswd

### delete and add users
htpasswd -bB users.htpasswd <username> <password>
htpasswd -D users.htpasswd <username>

### replace secret
oc create secret generic htpass-secret --from-file=htpasswd=users.htpasswd --dry-run=client -o yaml -n openshift-config | oc replace -f -

### delete, deleted users
oc delete user <username>

### delete identity object
oc delete identity my_htpasswd_provider:<username>


#RBAC

### Disable Self-provisioning
oc get clusterrolebinding -o wide |  grep -E 'ROLE|self-provisioner'

oc describe clusterrolebindings self-provisioners

oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth

oc get clusterrolebinding -o wide | grep -E 'ROLE|self-provisioner'

## as admin create project give right
oc new-project demo-2

oc policy add-role-to-user admin user1

oc adm groups new vm-group 

oc adm groups add-users vm-group user1

oc policy add-role-to-group view group

oc policy add-role-to-group edit group

oc get rolebindings -o wide | grep -v '^system:'

### Enable Self-provisioning
 oc adm policy add-cluster-role-to-group --rolebinding-name self-provisioners 
  self-provisioner system:authenticated:oauth

### Add local role
oc create role podview --verb=get --resource=pod -n blue

oc adm policy add-role-to-user podview user2 --role-namespace=blue -n blue

## cluster role
oc policy add-role-to-user view user1 

oc create clusterrole vmviewonly --verb=get --resource=vm

oc adm policy add-cluster-role-to-user vmview user1
