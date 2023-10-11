# 플라스크 애플리케이션을 쿠버네티스 클러스터에 배포

  

# LAB

  

아래 조건을 만족하는 플라스크 애플리케이션을 개발하고 쿠버네티스 클러스터에 배포해 보세요.

  

## Quest

  

1. 플라스크 애플리케이션은 /whoareyou 요청에 대해 작성자 이름, 호스트 이름, IP 주소를 반환
ex) 홍길동, hostname-deployment-7d4f978855-2kkw6, 10.0.0.4

2. 플라스크 애플리케이션을 구동하는 컨테이너 이미지의 이름은 whoami-flask:v1 으로 설정해 본인의 도커 허브에 등록

3. Deployment의 Replica는 5개로 설정해서 배포하기

4. LoadBalancer 타입의 서비스를 이용해서 Deployment를 연동

  



  



## **#1 가상환경 설정 및 flask 모듈 다운로드**

  

```bash

C:\kubernetes>  python  -m  venv  whoami-flask
C:\kubernetes>  cd  whoami-flask
C:\kubernetes\whoami-flask>  Scripts\activate
(whoami-flask) C:\kubernetes\whoami-flask> pip install flask
(whoami-flask) C:\kubernetes\whoami-flask> code .

```

  

## **#2 flask 애플리케이션 제작**

  

`C:\kubernetes\whoami-flask\**app.py**`

  

```python

from flask import Flask
import os
import socket

app = Flask(__name__)

@app.route('/whoareyou')

def  whoareyou():
	hostname = os.getenv('HOSTNAME', '')
	s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
	s.connect(("8.8.8.8", 53))
	ip = s.getsockname()[0]
	s.close()

	name = '홍길동'

	return name + " : " + hostname + " : " + ip

  

app.run(host="0.0.0.0",port=5000)

```

  

## **#3 flask 애플리케이션 테스트**

  

```cpp

(whoami-flask) C:\kubernetes\whoami-flask> **set HOSTNAME=testhostname** ⇐ 환경변수 설정
(whoami-flask) C:\kubernetes\whoami-flask> **python app.py**
* Serving Flask app 'app'
* Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
* Running on all addresses (0.0.0.0)
* Running on http://127.0.0.1:5000
* Running on http://172.20.132.191:5000
Press CTRL+C to quit

```

  

[https://lh6.googleusercontent.com/F5QiehAC3bthyukgiBlyZdb0sYbzYcp_GUc2gHNH-6YTrRXmFSMhwHJjbAcQa2jEMiAz2RLHKf5abeiZG6To7BaZlxFBKYxRDTnTwmbU5r2_rzHq0B3bAK7k0B20CB24Ar0vwXHN5JY5lzOTab8fHA](https://lh6.googleusercontent.com/F5QiehAC3bthyukgiBlyZdb0sYbzYcp_GUc2gHNH-6YTrRXmFSMhwHJjbAcQa2jEMiAz2RLHKf5abeiZG6To7BaZlxFBKYxRDTnTwmbU5r2_rzHq0B3bAK7k0B20CB24Ar0vwXHN5JY5lzOTab8fHA)

  

## **#4 requirements.txt 파일 생성**

  

```cpp

(whoami-flask) C:\kubernetes\whoami-flask> **pip freeze > requirements.txt**

  

(whoami-flask) C:\kubernetes\whoami-flask> **type requirements.txt**
blinker==1.6.3
click==8.1.7
colorama==0.4.6
Flask==3.0.0
itsdangerous==2.1.2
Jinja2==3.1.2
MarkupSafe==2.1.3
Werkzeug==3.0.0

```

  

## **#5 Dockerfile 작성**

  

`C:\kubernetes\whoami-flask\**Dockerfile**`

 
 
```docker

FROM python
WORKDIR /app
COPY app.py .
COPY requirements.txt .
RUN pip install -r requirements.txt
CMD ["python", "/app/app.py"]

```

  

## **#6 Docker 이미지 빌드**

  

```docker

(whoami-flask) C:\kubernetes\whoami-flask> **docker image build -t hwan515/whoami-flask:v1 .**
(whoami-flask) C:\kubernetes\whoami-flask> **docker image ls**
REPOSITORY TAG IMAGE ID CREATED SIZE
hwan515/whoami-flask v1 611c4415e509 12 seconds ago 1.03GB

```

  

## **#7 Docker 컨테이너 실행 및 테스트**

  

```docker

(whoami-flask) C:\kubernetes\whoami-flask> **docker container run --rm -p 5000:5000 hwan515/whoami-flask:v1**
* Serving Flask app 'app'
* Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
* Running on all addresses (0.0.0.0)
* Running on http://127.0.0.1:5000
* Running on http://172.17.0.2:5000
Press CTRL+C to quit

```

  

[https://lh4.googleusercontent.com/uIk8Yd7Dn2iaJwPMs3Gt2q-BJ1VXcfavBRfOZC5itZGn1GRYp7cdjh4I2t_famv0zZswpHZl02Mu_DiOqHlPHwAqBFszdoab89JtmhY6wKySTGlu1bL7qnT6N_NZAbS-xBLoC9Xj3iHFCs5oZvBreA](https://lh4.googleusercontent.com/uIk8Yd7Dn2iaJwPMs3Gt2q-BJ1VXcfavBRfOZC5itZGn1GRYp7cdjh4I2t_famv0zZswpHZl02Mu_DiOqHlPHwAqBFszdoab89JtmhY6wKySTGlu1bL7qnT6N_NZAbS-xBLoC9Xj3iHFCs5oZvBreA)

  

## **#8 Docker 이미지를 도커 허브에 등록**

  

```cpp

(whoami-flask) C:\kubernetes\whoami-flask> **docker image push hwan515/whoami-flask:v1**

```

  

## **#9** Deployment**와 Service 매니페스트 파일 작성**

  

`/home/vagrant/**whoami-flask.yaml**`

  

```yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: whoami-flask-deployment
spec:
  replicas: 5
  selector:
    matchLabels:
      app: whoami-flask
  template:
    metadata:
      name: whoami-flask-pod
      labels:
        app: whoami-flask
    spec:
      containers:
      - name: whoami-flask-container
        image: docker.io/myanjini/whoami-flask:v1
        ports:
        - containerPort: 5000
      imagePullSecrets:
      - name: regcred
---
apiVersion: v1
kind: Service
metadata:
  name: whoami-flask-service
spec:
  type: LoadBalancer
  ports:
  - name: whoami-flask
    port: 5000
    targetPort: 5000
  selector:
    app: whoami-flask

```

  

## **#10 디플로이먼트와 서비스 생성**

  

```cpp

vagrant@master-node:~$ **kubectl apply -f whoami-flask.yaml**
deployment.apps/whoami-flask-deployment created
service/whoami-flask-service created

vagrant@master-node:~$ **kubectl get deployment,pod,service**
NAME READY UP-TO-DATE AVAILABLE AGE

deployment.apps/whoami-flask-deployment 5/5  5  5  5m41s

  
NAME READY STATUS RESTARTS AGE
pod/whoami-flask-deployment-5b9659969-gmgsw 1/1 Running 0  5m41s
pod/whoami-flask-deployment-5b9659969-hhn4l 1/1 Running 0  5m41s
pod/whoami-flask-deployment-5b9659969-k9j9f 1/1 Running 0  5m41s
pod/whoami-flask-deployment-5b9659969-nrm56 1/1 Running 0  5m41s
pod/whoami-flask-deployment-5b9659969-zjh88 1/1 Running 0  5m41s

  
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
service/kubernetes ClusterIP 172.17.0.1 <none> 443/TCP 5d2h
service/whoami-flask-service LoadBalancer 172.17.24.141 <pending> 5000:32630/TCP 5m41s

```

  

- **로드밸런서가 존재하지 않기 때문에 Pending 상태가 지속

→ MetalLB 설치시 해결 가능**

### **#1 strictARP mode 활성화**

```bash

vagrant@master-node:~$ **kubectl get configmap kube-proxy -n kube-system -o yaml | \

> sed -e "s/strictARP: false/strictARP: true/" | \

> kubectl apply -f - -n kube-system**

Warning: resource configmaps/kube-proxy is missing the kubectl.kubernetes.io/last-applied-configuration annotation which is required by kubectl apply. kubectl apply should only be used on resources created declaratively by either kubectl create --save-config or kubectl apply. The missing annotation will be patched automatically.

configmap/kube-proxy configured

```

### **#2 MetalLB 설치**

```bash

vagrant@master-node:~$ **kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.11/config/manifests/metallb-native.yaml**
namespace/metallb-system created
customresourcedefinition.apiextensions.k8s.io/addresspools.metallb.io created
customresourcedefinition.apiextensions.k8s.io/bfdprofiles.metallb.io created
customresourcedefinition.apiextensions.k8s.io/bgpadvertisements.metallb.io created
customresourcedefinition.apiextensions.k8s.io/bgppeers.metallb.io created
customresourcedefinition.apiextensions.k8s.io/communities.metallb.io created
customresourcedefinition.apiextensions.k8s.io/ipaddresspools.metallb.io created
customresourcedefinition.apiextensions.k8s.io/l2advertisements.metallb.io created
serviceaccount/controller created
serviceaccount/speaker created
role.rbac.authorization.k8s.io/controller created
role.rbac.authorization.k8s.io/pod-lister created
clusterrole.rbac.authorization.k8s.io/metallb-system:controller created
clusterrole.rbac.authorization.k8s.io/metallb-system:speaker created
rolebinding.rbac.authorization.k8s.io/controller created
rolebinding.rbac.authorization.k8s.io/pod-lister created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:controller created
clusterrolebinding.rbac.authorization.k8s.io/metallb-system:speaker created
configmap/metallb-excludel2 created
secret/webhook-server-cert created
service/webhook-service created
deployment.apps/controller created
daemonset.apps/speaker created
validatingwebhookconfiguration.admissionregistration.k8s.io/metallb-webhook-configuration created

vagrant@master-node:~$ **kubectl get all -n metallb-system**
NAME READY STATUS RESTARTS AGE
pod/controller-64f57db87d-w9sz9 1/1 Running 0 62s
pod/speaker-2bhqq 1/1 Running 0 62s
pod/speaker-shx44 1/1 Running 0 62s
pod/speaker-sns7n 0/1 Running 0 62s

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
service/webhook-service ClusterIP 172.17.8.7 <none> 443/TCP 62s

NAME DESIRED CURRENT READY UP-TO-DATE AVAILABLE NODE SELECTOR AGE
daemonset.apps/speaker 3 3 2 3 2 kubernetes.io/os=linux 62s

NAME READY UP-TO-DATE AVAILABLE AGE
deployment.apps/controller 1/1 1 1 62s

NAME DESIRED CURRENT READY AGE
replicaset.apps/controller-64f57db87d 1 1 1 62s

```

### **#3 LoadBalancer 서비스에 할당할 IP 대역을 정의

⇒ VirtualBox Hosy-Only 네트워크 관리자의 설정을 확인**

[https://lh5.googleusercontent.com/lpFMV8vVMfGzgIVGkb997TQELMTBNpAxJdhLhmr6shBZBY_j0ApXrDCjnbea4JwMzWTHm1JxA8EPQpvxUS5DT4NjUPVz8c3lFbnV8OnzIYG1Xmwec8qm7DBrceF6Fn3Cy2MEGRDYhz-ugMd3tvl1eA](https://lh5.googleusercontent.com/lpFMV8vVMfGzgIVGkb997TQELMTBNpAxJdhLhmr6shBZBY_j0ApXrDCjnbea4JwMzWTHm1JxA8EPQpvxUS5DT4NjUPVz8c3lFbnV8OnzIYG1Xmwec8qm7DBrceF6Fn3Cy2MEGRDYhz-ugMd3tvl1eA)

참고 ⇒ [https://metallb.universe.tf/configuration/](https://metallb.universe.tf/configuration/)

`/home/vagrant/**routing-config.yaml` 생성하기**

```yaml

apiVersion: metallb.io/v1beta1
kind: IPAddressPool
metadata:
name: first-pool
namespace: metallb-system
spec:
addresses:
- 10.0.0.30-10.0.0.254

```

```bash

# 적용

vagrant@master-node:~$ **kubectl apply -f routing-config.yaml -n metallb-system**
ipaddresspool.metallb.io/first-pool created

# 확인

vagrant@master-node:~$ **kubectl get IPAddressPool -n metallb-system**
NAME AUTO ASSIGN AVOID BUGGY IPS ADDRESSES
first-pool true false ["10.0.0.3-10.0.0.254"]

```

  

---

  

**설치후**

  

```cpp

vagrant@master-node:~$ **kubectl get deployment,pod,service**
NAME READY UP-TO-DATE AVAILABLE AGE
deployment.apps/whoami-flask-deployment 5/5  5  5  9s

  

NAME READY STATUS RESTARTS AGE
pod/whoami-flask-deployment-5b9659969-b678t 1/1 Running 0  9s
pod/whoami-flask-deployment-5b9659969-lvm8l 1/1 Running 0  9s
pod/whoami-flask-deployment-5b9659969-pbg5k 1/1 Running 0  9s
pod/whoami-flask-deployment-5b9659969-rqvqs 1/1 Running 0  9s
pod/whoami-flask-deployment-5b9659969-sdlcl 1/1 Running 0  9s

  

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
service/kubernetes ClusterIP 172.17.0.1 <none> 443/TCP 5d2h
service/whoami-flask-service LoadBalancer 172.17.40.144  10.0.0.30  5000:31637/TCP 9s

```

  

## **#11 로드밸런서 IP로 접근했을 때 다섯 개의 파드로 요청이 분배되는 것을 확인**

  

```bash

vagrant@master-node:~$  **wget  -q  -O  -  http://10.0.0.30:5000/whoareyou**
홍길동  :  whoami-flask-deployment-5b9659969-b678t  :  172.16.158.9

vagrant@master-node:~$  **wget  -q  -O  -  http://10.0.0.30:5000/whoareyou**
홍길동  :  whoami-flask-deployment-5b9659969-lvm8l  :  172.16.158.11

vagrant@master-node:~$  **wget  -q  -O  -  http://10.0.0.30:5000/whoareyou**
홍길동  :  whoami-flask-deployment-5b9659969-pbg5k  :  172.16.158.10

vagrant@master-node:~$  **wget  -q  -O  -  http://10.0.0.30:5000/whoareyou**
홍길동  :  whoami-flask-deployment-5b9659969-rqvqs  :  172.16.87.251

vagrant@master-node:~$  **wget  -q  -O  -  http://10.0.0.30:5000/whoareyou**
홍길동  :  whoami-flask-deployment-5b9659969-sdlcl  :  172.16.87.250

```
