
kubernetes acccount

https://kubernetes.io/ko/docs/reference/access-authn-authz/service-accounts-admin/

사용자 account - 사람을 위한것, 클러스트 전체에 영향을 미친다
서비스 account - 파드에서 실행된 프로세스를 위한것, 네임스페이스에 할당

service account(sa)

kubectl get service --as system:serviceaccount:example:myuser 
<= example 에서 myuser 권한으로 서비스 목록 출력

* serviceaccount 의 사용자 이름
system:serviceaccount::

* 특정 사용자 권한으로 kubectl command 실행 
ex) kubectl get pods  --as system:serviceaccount:myns:myuser

서비스 어카운트에 적절한 권한이 없으면 쿠버네티스 리소스를 제대로 사용할수 없다.

서비스 어카운트에 권한부여방법

kubernetes RBAC
- 역할기반으로 쿠버네티스의 권한 관리
사용자와 역할의 조합으로 사용자에게 특정권한을 부여

kubernetes rolebinding
- 사용자에게 롤을 할당

role/clusterrole

* clusterrole 은 namespace 영향을 받지 않는다.
1. role 을 사용하여 권한부여

2. cluster role 을 사용하여 권한부여 

kubectl get clusterrole

serviceaccount 생성
```sh
cat sa.yaml
```
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: myuser
```
```sh
$ kubectl describe serviceaccounts myuser

Name:                myuser
Namespace:           myns
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              <none>
Events:              <none>
```
[vagrant@master test]$
-------------------------------------------------------------------------
pod 사용권한 설정
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: examplens
  name: use-pod
rules:
- apiGroups: [""] # "" indicates the core API group , "" <- core API group
  resources: ["pods"]
  verbs: ["create", "get", "watch", "list"]
```
```sh
$ kubectl describe role use-pod

Name:         use-pod
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 []              [create get watch list]
```

get: for individual resources,  
list : for collections, including full object content  
watch: for watching individual resources or collecton of resources  
----------------------------------------------------------------------------
```yaml
kind: RoleBinding
metadata:
  name: use-pod
  namespace: examplens
subjects:
- kind: ServiceAccount
  name: user1-sa # "name" is case sensitive
  namespace: examplens
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: use-pod # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```
```sh
$ kubectl describe rolebindings.rbac.authorization.k8s.io use-pod

Name:         use-pod
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  use-pod
Subjects:
  Kind            Name    Namespace
  ----            ----    ---------
  ServiceAccount  myuser  myns
```
```sh 
# 생성가능 : 해당 유저로 생성하는 것
$ kubectl run nginx --image=nginx --as system:serviceaccount:myns:myuser

pod/nginx created
```
```sh
# 삭제권한은 주지 않았으므로 삭제가 안됨
$ kubectl delete pods nginx --as system:serviceaccount:myns:myuser

Error from server (Forbidden): pods "nginx" is forbidden: User "system:serviceaccount:myns:myuser" cannot delete resource "pods" in API group "" in the namespace "myns"
```

--------------------------------------------------------
```sh
$ cat service-clusterrole.yaml
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  namespace: examplens
  name: use-all-service
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["services"]
  verbs: ["get","list"]
```

---------------------------------------------------------
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: use-all-service
  namespace: examplens
subjects:
- kind: ServiceAccount
  name: user2-sa # "name" is case sensitive
  namespace: examplens
roleRef:
  kind: ClusterRole #this must be Role or ClusterRole
  name: use-all-service # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

