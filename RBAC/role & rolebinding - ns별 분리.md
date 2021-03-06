# RBAC

### [](https://github.com/jjsair0412/kubernetes_info/blob/main/RBAC/RBAC.md#reference)reference

[https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user](https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user)

## [](https://github.com/jjsair0412/kubernetes_info/blob/main/RBAC/RBAC.md#1-create-user)1. Create user

**1-1. 인증서 생성**

```
# key파일 생성 
$ openssl genrsa -out myuser.key 2048 


# key파일을 기준으로 csr파일 생성
$ openssl req -new -key myuser.key -out myuser.csr -subj "/CN=myuser"  
```

**1-2. kubernetes에 생성한 유저 등록**

-   request에 들어갈 인증서 정보 생성 명령어

```
cat myuser.csr  | base64 | tr -d "\n"
```

```
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser # csr name
spec:
  request: # 방금전 만들어줬던 인증서를 확인해서 인증서 내용을 넣어 주어야 한다.
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400 # one day . 유효기간을 의미.
  usages:
  - client auth
```

**1-3. apply**

```
$ kubectl apply -f myuser.yaml
```

**1-4. 등록상태 확인**

```
$ ﻿kubectl get csr
```

-   pending 상태 인 것을 볼 수 있음. 따로 승인을 해주어야 한다.

```
$ kubectl certificate approve myuser
```

등록상태를 재 확인 하면 ,  **Approved,Issued**  상태로 변환 된 것을 볼 수 있다.

```
$ ﻿kubectl get csr
```

**1-5. crt 파일 생성**

```
$ kubectl get csr myuser  -o jsonpath='{.status.certificate}'| base64 -d > myuser.crt
```

## [](https://github.com/jjsair0412/kubernetes_info/blob/main/RBAC/RBAC.md#2-create-role)2. Create Role

**2-1. Create yaml file**

```
apiVersion: rbac.authorization.k8s.io/v1 
kind: Role 
metadata: 
  namespace: default 
  name: myuser-role 
rules:  
- apiGroups: [""] 
  verbs: ["create","get","list","update"]
  resources: ["*"]

```

**apiGroups**

-   **어디에 해당 Role을 지정할것인가**를 정의
-   비워두면 모든 리소스가 대상이 됨

**verbs**

-   **지정한 api그룹에 어떤 권한을 부여할것인가**를 정의

### [](https://github.com/jjsair0412/kubernetes_info/blob/main/RBAC/RBAC.md#verb%EC%9D%98-%EC%A2%85%EB%A5%98)verb의 종류
| kind | description |
|--|--|
|**create**| 새로운 리소스 생성 |
| **get** | 개별 리소스 조회 |
|**list**|여러건의 리소스 조희|
|**update**|기존 리소스 내용 전체 업데이트|
|**delete**|개별 리소스 삭제|
|**deletecollection**|여러 리소스 삭제|
|**patch**|리소스 수정|

## [](https://github.com/jjsair0412/kubernetes_info/blob/main/RBAC/RBAC.md#3-create-rolebinding)3. Create RoleBinding

** 3-1 RoleBinding 생성 **
```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: null
  name: myuser-rolebinder
  namespace: default # namespace별로 구분
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: myuser-role # role 이름
subjects:
- kind: User
  name: myuser # user 이름
```

## [](https://github.com/jjsair0412/kubernetes_info/blob/main/RBAC/RBAC.md#4-config-file-%EB%93%B1%EB%A1%9D)4. Config file 등록

**4-1. kubectl 명령어를 통해서 config 정보를 확인**

```
$ ﻿kubectl config view
```

**4-2. 아래의 형식을 통해서 kubeconfig 파일에 user를 등록**

```
$ ﻿kubectl config set-credentials myuser --client-certificate=myuser.crt --client-key=myuser.key --embed-certs=true
```

**4-3. context 추가**

- 명령어 옵션으로 **namespace** 를 주면 , kubectl 명령어를 출력시킬 namespace를 고정시킬 수 있다.
```
﻿$ kubectl config set-context myuser --cluster=cluster.local --user=myuser --namespace=jenkins 
```

**4-4. context 변경**

```
$ ﻿kubectl config use-context myuser 
```
## 5. user-role 삭제 절차 (완전 삭제)
**5-1. context 삭제**
- config 파일에 등록한 context 정보를 삭제 합니다.
```
$ kubectl config delete-context myuser
```
**5-2. user 삭제**
- config 파일에 등록한 user 정보를 삭제 합니다.
```
$ kubectl config delete-user myuser
```
**5-3. rolebinding 삭제**
- rolebinding을 삭제 합니다.
```
$ kubectl delete rolebinding myuser-rolebinding
```
**5.4. role 삭제**
- role을 삭제 합니다.
```
$ kubectl delete role myuser-clusterrole
```