# 07 EXPOSING APPLICATIONS

### 학습 목표

- 서비스를 사용하여 애플리케이션 노출
- ClusterIP 서비스 
- NodePort 서비스 
- LoadBalancer 서비스 
- ExternalName 서비스 
- Ingress Controller 이해

### 1 사전 지식

#### pod와 service

- pod는 컨트롤러에 의해 관리되기 때문에 한군데에 고정되서 떠 있지 않고, 클러스터 내를 옮겨다님

- 이 과정에서 노드를 옮기면서 실행되기도 하고, 클러스터 내의 pod IP가 변경되기도 함

- 동적으로 변하는 pod에 고정된 방법으로 접근하기 위해 **kubernetes 서비스(service) 사용**

- 서비스 사용하면,  pod가 클러스터 내의 어디에 있든지 상관없이 고정된 주소를 이용해서 접근 가능

- 또한, 클러스터 외부에서 pod에 접근하는 것도 **서비스(service)**를 통해 가능

  

  ![image](https://user-images.githubusercontent.com/38436013/109374483-df639500-78f8-11eb-8483-156070bd26e8.png)

- 서비스는 pod가 옮겨 갔을때 서비스가 자동으로 새로 뜬 pod를 지켜보기 때문에 사용자는 서비스만 바라보고 있으면 문제없이 pod 사용 가능 

### 2 서비스

- pod 집합에서 실행중인 애플리케이션을 네트워크 서비스로 노출하는 추상화 방법
- 논리적 pod set을 정의하고, 외부 트래픽 노출, 로드밸런싱 그리고 pod들에게 접근할 수 있는 정책
- YAML 또는 JSON을 이용하여 정의

#### 서비스 템플릿

~~~
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  type: ClusterIP
  clusterIP: 10.0.10.10
  selector:
    app: MyApp
  ports:
  - protocol: TCP
    port: 80
    targetPort: 9376
~~~

- spec.type으로 서비스 타입 지정, 이것을 지정하지않으면 ClusterIP타입
- spec.selector에서는 서비스와 연결할 pod에 지정된 라벨을 지정
- spec.ports는  서비스가 포트를 외부에 여러개를 한꺼번에 제공할때 ports하위에 값을 적어줌

#### 헤드리스 서비스

- **what?** ClusterIP를 None으로 설정하면 ClusterIP가 없는 서비스 생성 => 헤드리스
- **when?** 로드밸런싱이 필요없거나 단일 서비스 IP가 필요 없는 경우

##### 서비스 YAML

- 셀렉터 있는 서비스

  - 엔드포인트 레코드 생성

  ~~~
  apiVersion: v1
  kind: Service
  metadata:
    name: my-service
  spec:
    selector:
      app: MyApp
    ports:
      - protocol: TCP
        port: 80
        targetPort: 9376
  ~~~

- 셀렉터 없는 서비스

  - 나중에 엔드포인트 추가하거나, 다른 서비스로 서비스 지정하고 싶을때

  - 엔드포인트 레코드 생성X

  ~~~
  apiVersion: v1
  kind: Service
  metadata:
    name: my-service
  spec:
    ports:
    - protocol: TCP
        port: 80
        targetPort: 9376
  ~~~

  - 엔드포인트 오브젝트를 수동으로 추가 ->  서비스를 실행 중인 네트워크 주소 및 포트에 서비스를 수동으로 매핑 가능

  ~~~
  apiVersion: v1
  kind: Endpoints
  metadata:
    name: my-service
  subsets:
    - addresses:
        - ip: 192.0.2.42
      ports:
        - port: 9376
  ~~~



### 3 서비스 유형

- 일부 애플리케이션을 클러스터 외부에 ip 노출하고 싶을때 서비스 유형을 지정 가능

#### ClusterIP 

클러스터 내부 IP에 대해 서비스 노출, 클러스터 내에서만 서비스 접근 가능(클러스터 내부의 노드나 pod에서 이 ClusterIP를 이용해서 이 서비스에 연결된 pod에 접속 가능)

#### NodePort

서비스에 지정된 port를 각 노드에 할당하는 방식

클러스터 IP 뿐 아니라, 모든 노드의 port를 통해서도 접근이 가능

**NodePort의 사용으로 클러스터 내부 , 외부 모두 접근 가능**

`<NodeIP>:<NodePort>`  이용하여 클러스터 외부로부터 NodePort 서비스 접근

~~~
apiVersion: v1
kind: Service
metadata:
  name: hello-node-svc
spec:
  selector:
    app: hello-node
  type: NodePort
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: 8080
      nodePort: 30036  
~~~

- type 필드를 NodePort로 설정하면 이 서비스가 NodePort임

- nodePort를 30036 설정하면, 클러스터 ip의 80포트로도 접근 가능하지만,

- 모든 노드의 30036포트로도 접근 가능

  ![image](https://user-images.githubusercontent.com/38436013/109307421-f9af5b80-7883-11eb-84ef-af9c26d00f92.png)

#### LoadBalancer

AWS, GCP 같은 클라우드 서비스를 사용할때 사용가능한 옵션

pod를 클라우드에서 제공하는 로드밸런서와 연결해서 그 로드밸런서의 ip를 이용해서 클러스터 외부에서 접근 가능(외부용 로드밸런서 생성해서, 그 외부 ip를 통해 외부에서 접근 가능)

#### ExternalName

외부서비스를 쿠버네티스 내부에서 호출하고자 할때 사용(rds, cloudsql 등)

클러스터 내의 pod들은 클러스터 ip 대역 밖의 서비스를 호출하려면 복잡한 NAT 설정이 필요한데,

이것을 ExternalName이 해결!!

셀렉터 필요 x

~~~
kind: Service
apiVersion: v1
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
~~~

- 서비스 타입을 ExternalName 설정,
- 주소를 DNS로  my.database.example.com으로 설정,
- my-service로 들어오는 모든 요청을 my.database.example.com 으로 포워딩(일종의 프록시 역할)

## 4 kube-proxy

- 서비스를 클러스터 내부 IP로 연결 요청을 전달해주는 역할
- kubernetes 클러스터의 각 노드마다 실행(각 노드에 정적 pod로 배포됨)
- kube-proxy가 네트워크 관리하는 방법(3가지)
  - userspace : 초기모드
  - iptables : 현재 기본모드
  - ipvs : 앞으로 전향될 모드

#### userspace 모드

![image](https://user-images.githubusercontent.com/38436013/109373439-1edab300-78f2-11eb-96cf-8386e29f28a4.png)

- 동작
  1. 클라이언트 -> 서비스IP로 요청
  2. 서비스IP의 iptables -> kube-proxy 가 요청 받음
  3. kube-proxy가 라운드로빈 방식을 통해 요청을 pod로 나눠줌(연결)

#### iptables 모드

- userspace모드와 다른점 : kube-proxy는 iptables를 관리하는 역할만! **클라이언트로부터 트래픽 안받아!**

- 동작

  1. 클라이언트 -> clusterIP로 요청 
  2. clusterIP의 iptables -> pod로 요청 전달됨

- 장점 

  - userspace 모드보다 빠르다

    - userpace 모드에서 라운드로빈으로 요청 처리하는데, 비정상 pod로의 요청 실패하면 다시 다른 pod로 연결 재시도 

    - iptables 모드에서는 pod 하나로의 요청 실패면, 재시도 안하고 그냥 실패 

    - 이것을 방지하기 위해  readiness probe 설정

![image-20210226161353086](C:\Users\yujin\AppData\Roaming\Typora\typora-user-images\image-20210226161353086.png)

#### ipvs 모드

- ipvs(IP Virtual Server) 모드는 리눅스 커널에 있는 L4 로드밸런싱 기술로 Netfilter
- ipvs는 커널스페이스에서 작동하고 데이터 구조를 해시테이블로 저장해서 가지고 있기 때문에 iptables보다 빠르고 좋은 성능
- rr(round-robin), lc(least connection), dh(destination hashing), sh(source hashing), sed(shortest expected delay), nq(never queue)등의 다양한 로드밸런싱 알고리즘

![image](https://user-images.githubusercontent.com/38436013/109373939-23549b00-78f5-11eb-855a-62389974c093.png)

## 5 Ingress Controller

#### Ingress

- Ingress : 클러스터 외부 -> 내부로 접근하는 요청 처리 규칙 모음
- 부에서 접근가능한 URL을 사용할 수 있게 하고, 트래픽 로드밸런싱도 해주고, SSL 인증서 처리도 해주고, 도메인 기반으로 가상 호스팅을 제공
- ingress가 동작하게 해주는 것 -> **Ingress Controller** 
- 클라우드 서비스를 사용하지 않는 경우,  Ingress Controller와 직접 Ingress와 연동해 주어야 함
- 이때 가장 많이 사용하는 ingress-nginx(https://github.com/kubernetes/ingress-nginx)
- Ingress 용 yaml 템플릿

~~~
piVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /foos1
        backend:
          serviceName: s1
          servicePort: 80
      - path: /bars2
        backend:
          serviceName: s2
          servicePort: 80
  - host: bar.foo.com
    http:
      paths:
      - backend:
          serviceName: s2
          servicePort: 80
~~~

- foo.bar.com으로 요청->  /foos1이면 서비스 s1으로 연결,   /bars2이면 서비스 s2쪽으로 연결
- 또 라든 호스트명인 bar.foot.com도 서비스 s2로 연결
- 결론은 ingress사용하면 외부에서 들어오는 요청 간단히 처리

#### Ingress Controller : nginx

- Ingress는 설정일 뿐 동작하게 하는 것이 Ingress Controller

- Ingress Controller를 통해서 트래픽이 전달될때 중간에 서비스를 거치지 않고 직접 포드로 연결되는 구조

- 동작

  별도읜 NodePort없이 IngressController에 접근가능하게 되고, 다시 Pod로 접근 가능하게 되기 때문에 빠른 속도 성능 가질 수 있음

  ![image](https://user-images.githubusercontent.com/38436013/109375574-2ce40000-7901-11eb-80a3-e8fa4a9cfe4e.png)



#### 서비스와 Ingress

공통점 : 서비스도 외부에서 내부로 접근 가능

차이점 

 	1.  서비스가 너무 많아질 경우, 독립적으로 사용하면 관리의 어려움과 리소스 낭비 -> 유연성 제공
 	2.  낮은 번호의 포트를 애플리케이션에 노출 가능 -> iptables 는 관리 어려움

## 6 서비스 메시(Service Mesh)

- 서비스간 통신 제어하고 관리하도록 마이크로서비스를 위한 인프라 계층
- 기존 서비스에서의 호출이 직접 방식이라면, 서비스 메시는 인프라 계층의 porxy 이용

#### Mesh Network

![image-20210227134954383](C:\Users\yujin\AppData\Roaming\Typora\typora-user-images\image-20210227134954383.png)

- 개별 proxy는 각 서비스와 함께 실행해서 sidecar 라고 부르는데
- sidecar proxy들이 모여 mesh network 형성

#### 서비스 메시로 인한 이점

- MSA 시스템은 수십개의 마이크로서비스들로 구성되어 있는데, 서비스간 통신 매우 복잡-> 서비스 메시 없이는 장애 통제 X
- 개발자들이 비지니스 로직에 집중 장점

#### API GW vs Service Mesh

- API GW : 네트워크 외부 트래픽 수락하여 내부로 배포
- 서비스 메시 :  네트워크 내부에서 트래픽 라우팅, 관리
- 두개는 함께 작동, 외부 트래픽이 API GW 통해 라우팅 된후 서비스 메시로 라우팅

##  7 용어

#### 라운드로빈

- os의 scheduling 기법중 대표적인..

- 스케줄링(scheduling) : 처리할 일들의 순서를 정하는 일

  cpu 이용률과 처리율을 높이는 것이 목표, (즉, cpu가 쉬지 않고 처리할 수 있게 효율적인 계획 잡아주는일)

- 라운드로빈 : 시분할 시스템을 위해 설계된 선점형 스케줄링

  순서대로 시간단위르 cpu를 할당하는 방식

  위에서는 pod들의 순서대로 요청을 처리해 주겠지? (종료되거나 비정상 pod들은 건너뛰고 ??)
