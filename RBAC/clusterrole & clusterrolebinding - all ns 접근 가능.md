
# RBAC

### [](https://github.com/jjsair0412/kubernetes_info/blob/main/RBAC/RBAC.md#reference)
- 참고 링크 : https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/#normal-user
## [](https://github.com/jjsair0412/kubernetes_info/blob/main/RBAC/RBAC.md#1-create-user)1. Create Other User
- 전체 Namespace에 접근이 가능하지만 , 리소스에 대해 delete를 할 수 없게끔 생성한 dev-user

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
  request: 
  signerName: kubernetes.io/kube-apiserver-client
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
$ kubectl get csr myuser -o jsonpath='{.status.certificate}'| base64 -d > myuser.crt
```

## [](https://github.com/jjsair0412/kubernetes_info/blob/main/RBAC/RBAC.md#2-create-role)2. Create clusterrole

**2-1. Create yaml file**

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: myuser-clusterrole
rules:
- apiGroups:
  - "*"
  resources:
  - '*'
  verbs:
  - get



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

## [](https://github.com/jjsair0412/kubernetes_info/blob/main/RBAC/RBAC.md#3-create-rolebinding)3. Create clusterrolebinding

** 3-1 RoleBinding 생성 **
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: myuser-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: myuser-clusterrole
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: myuser

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

- 명령어 옵션으로 **namespace** 를 주면 , kubectl 명령어를 출력 시킬 namespace를 고정 할 수 있다.
```
﻿$ kubectl config set-context myuser --cluster=cluster.local --user=myuser
```

**4-4. context 변경**

```
$ ﻿kubectl config use-context myuser
```


**4-5. 현재 사용중인 context 확인**

```
$ kubectl config current-context
```
## 5. kubeconfig에서 내가 필요한 user정보만 꺼내오는 방법
```
$ kubectl config get-contexts
```

```
$ kubectl config view --context={context_name} --minify --flatten

# 사용 예
$ kubectl config view --context=myuser --minify --flatten
```