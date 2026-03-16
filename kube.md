## Kubernetes

- 전체적인 구조

```

                 Internet
                     │
                 Ingress
                     │
                 Service
                     │
            ┌────────┴────────┐
            ▼                 ▼
          Pod               Pod
        (container)       (container)
            │                 │
      ┌───────────── Cluster ─────────────┐
      │                                   │
      │        Control Plane              │
      │                                   │
      │   Nodes running workloads         │
      │                                   │
      └───────────────────────────────────┘
```

- EOS 발생했을 때 docker로 이미지를 따로 받아서 사용하면 가능함
- 외부 접근을 하려면 service, ingress를 설정해줘야 가능 
- 클러스터 → 노드 → 파드 → 컨테이너
	- cluster
		- 가장 큰 단위
		- node를 포함하는 전체 영역
	- node
		- 실제로 컨테이너가 실행되는 서버
		- kubelet
			- Control Plane Node와 Worker Node를 연결
				- 컨트롤 플레인(control plane)이 전체를 관리	
					- API Server
						- 쿠버네티스 모든 요청의 입구
					- Scheduler
						- Pod가 어떤 Node에서 실행될지 결정
					- Controller Manager
						- 클러스터 상태 유지
					- etcd
						- 쿠버네티스 상태 저장 DB
		- kube-proxy
			- 클러스터 네트워크 관리
		- Container Runtime
			- 컨테이너 실행
	- pod
		- 쿠버네티스에서 가장 작은 배포 단위
	- service
		- 용도
			- 로드밸런서 (LoadBalancer)
			- 외부 통신 (NodePort)
			- 내부 통신 (ClusterIP)
		- 역할
			- Load Balancing 
			- Pod discovery 
			- Stable IP 제공 
				- Pod는 IP가 계속 바뀌기 때문에 직접 접근하지 않음 
		- ip 구조: [svc].[ns]:[port]
	- namespace
		- 하나의 클러스터 안에서 리소스를 나누는 단위
		- 클러스터 안의 리소스를 논리적으로 분리하는 공간 (= 같은 네트워크 안에서 프로젝트 폴더 나눈 것)
		- 관리/권한/리소스 분리
		- 구성 요소
			- Pod
			- Service
			- Deployment
			- ConfigMap
			- Secret
			- Ingress 등
	- replicaset
		- pod 개수 지정
		- yml 파일 수정 시 apply를 한다고 해서 바로 적용되는 것이 아니라 pod가 새롭게 생성될 때 적용됨
		- 사용자가 사용 중에 중지되는 문제 발생 -> deployment 이용하여 자동으로 처리함


## 명령어
- pod 자리에 nodes/ns/deploy/svc/ingress/ingressClass/cm/secret 대체 가능

**get**
```
kubectl get pod --selector [label name]=[label value] --show-labels -L [label name1,label name2,...] -n [ns] -o [options]

--selector: 특정 label 가져오기
--show-labels: label 확인
-o
	- wide: ip, node, nominated node, readiness gates 정보 확인
	- yaml
-n: ns 지정

label 적용)
kubectl get pod -L app,env 

endpoint 조회)
kubectl get endpoints -n delivery

service 조회)
kubectl get svc -n delivery 

deploy 조회)
kubectl get deploy -n delivery

ingress 조회)
kubectl get ingress -n delivery

configmap 조회)
kubectl get cm -n delivery

secret 조회)
kubectl get secret -n argocd
kubectl get secret -n argocd argocd-initial-admin-secret -o jsonpath="{.data.password}"

환경변수 조회)
kubectl exec myapp-6489776d8c-crf6t -n delivery -- env

yaml 확인)
kubectl get cm cm-basic -o yaml -n delivery
```
---

**delete**
```
kubectl delete pod [pod] --all
--all: 모든 pod 삭제

모든 pod 삭제)
kubectl delete pod --all

kubectl delete -f [path]
-f: 파일 경로
```
---

**apply**
```
kubectl apply -f [yml]

경로를 통한 설치)
kubectl apply -f ./pod-basic.yml

nginx ingress 설치)
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.1/deploy/static/provider/cloud/deploy.yaml
```
---

**describe**
```
kubectl describe pod [pod] | findstr [str]

myapp pod 상세)
kubectl describe pod myapp
```
---

**label**
```
kubectl label pod [pod] [label name]=[label value] --overwrite [label name]-
--overwrite: 수정 시 사용

label 지정)
kubectl label pod label-app-4 app=app-4
kubectl label pod label-app-4 env=111 group=222

app, env label 삭제)
kubectl label pod label-app-4 app- env-
```
---

**scale**
```
kubectl scale rs [rs] --replicas [num]

replaicas 0)
kubectl scale rs rs-basic --replicas 0
- get pod => 존재하지 않음
- get rs => 0의 값으로 유지
```
---
**exec**
```
kubectl exec -it [pod] -- [cmd]

도커에 접근하여 bash 실행)
kubectl exec -it myapp -- bash

html 내용 수정)
kubectl exec -n delivery back-deploy-79db967b9f-kj2x9 -- sh -c 'echo "web-test: $(hostname -i)" > /usr/share/nginx/html/index.html'
kubectl exec -n delivery front-deploy-d6c6d85bc-pz6g9 -- sh -c 'echo echo "<h2>REACT</h2>boot ip: $(hostname -i)" > /usr/share/nginx/html/index.html'
```
---

## argo

```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl get pod -n argocd

kubectl get svc -n argocd
NAME                                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
argocd-applicationset-controller          ClusterIP   10.96.228.186   <none>        7000/TCP,8080/TCP            43m
argocd-dex-server                         ClusterIP   10.96.160.52    <none>        5556/TCP,5557/TCP,5558/TCP   43m
argocd-metrics                            ClusterIP   10.96.162.156   <none>        8082/TCP                     43m
argocd-notifications-controller-metrics   ClusterIP   10.96.254.230   <none>        9001/TCP                     43m
argocd-redis                              ClusterIP   10.96.135.153   <none>        6379/TCP                     43m
argocd-repo-server                        ClusterIP   10.96.160.236   <none>        8081/TCP,8084/TCP            43m
argocd-server                             ClusterIP   10.96.191.231   <none>        80/TCP,443/TCP               43m
argocd-server-metrics                     ClusterIP   10.96.193.250   <none>        8083/TCP                     43m
front-svc                                 ClusterIP   10.96.44.142    <none>        10000/TCP                    4m17s

kubectl port-forward svc/argocd-server -n argocd 12345:443 
```
---

## configmap
- 환경 변수 설정
----

## yml
- pod
- replicaset
- namespace
- service
- deploy
- ingress
- configmap
- secret

**pod**
```
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp-container
    image: nginx:latest
    resources:
      limits:
        memory: "100Mi"
        cpu: "500m"
    ports:
    - containerPort: 80
```
---

**replicaset**
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: rs-basic
spec:
  replicas: 5
  selector:
    matchLabels:
      app: value
  template:
    metadata:
      labels:
        app: value
    spec:
      containers:
        - name: nginx-con
          image: nginx:latest
          resources:
            limits:
              memory: "100Mi"
              cpu: "500m"

```
---

**namespace**
```
apiVersion: v1
kind: Namespace
metadata:
  name: delivery
```
---

**deploy**
```
# env 개별적 설정
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  namespace: delivery
spec:
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: nginx:latest
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
        env:
          - name: DB_USERNAME
            valueFrom: 
              configMapKeyRef:
                name: cm-basic
                key: DB_USER
          - name: DB_ADDRESS
            valueFrom:
              configMapKeyRef:
                name: cm-basic
                key: DB_URL
```
```
# env 한번에 설정
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cm-deploy
  namespace: delivery
spec:
  selector:
    matchLabels:
      app: cm-deploy
  template:
    metadata:
      labels:
        app: cm-deploy
    spec:
      containers:
      - name: myapp
        image: nginx:latest
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
        ports:
        - containerPort: 80
        envFrom:
          - configMapRef:
              name: cm-basic
```
---

**service**
```
apiVersion: v1
kind: Service
metadata:
  name: front-svc
  namespace: delivery
spec:
  type: ClusterIP
  selector:
    app: front-app
  ports:
  - port: 10000
    targetPort: 80
```
---

**ingress**
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myingress
  namespace: delivery
  annotations: 
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
    # - host: <Host>
    - http:
        paths:
          - pathType: Prefix
            path: /member
            backend:
              service:
                name: front-svc
                port:
                  number: 10000

```
---

**configmap**
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-basic
  namespace: delivery
data:
  DB_USER: admin-cm-test
  DB_URL: localhost-cm-test

설정한 환경변수 확인)
> kubectl exec myapp-685699855f-zspgb -n delivery -- env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=myapp-685699855f-zspgb
NGINX_VERSION=1.29.6
NJS_VERSION=0.9.6
NJS_RELEASE=1~trixie
ACME_VERSION=0.3.1
PKG_RELEASE=1~trixie
DYNPKG_RELEASE=1~trixie
DB_ADDRESS=localhost-cm-test
DB_USERNAME=admin-cm-test
```
---

**secret**
```
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: <Password>
```
---