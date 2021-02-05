 # 03 BUILD_1

## 목표

- 런타임 및 컨테이너 옵션
- 애플리케이션 컨테이너화
- 로컬 저장소를 호스팅
- 다중 컨테이너 포드를 배포
- readinessProbes 구성
- livenessProbes 구성

## BUILD

### Docker

- 컨테이너화된 애플리케이션의 실행
- 도커를 사용하면 애플리케이션을 쉽게 컨테이너화, 배포 및 소비 가능
- Docker Hub => 여러 아키텍처에서 개별 생성 이미지 다운 및 배표 가능
- kubernetes는 기본적으로 docker 엔진을 사용해 컨테이너 실행
- docker는 swarm 오케스트레이션 기능 추가 -> CRI-O 개방형 도구 사양 증가

### CRI(Container Runtime Interface)

- 목표 => 컨테이너 런타임을 kubelet과 쉽게 통합 

  - kubelet : 클러스터의 각 노드에서 실행되는 에이전트, 파드에서 컨테이너 동작을 관리

  - 1) 도커 기반 쿠버네티스 동작

    - kulelet이 명령받으면 , docker runtime이 도커 컨테이너 생명주기 관리

    ![image](https://user-images.githubusercontent.com/38436013/106985264-77fc6e80-67ac-11eb-97fe-e54ad359438d.png)

    

  - 2) 도커 외에 여러 컨테이너 등장

    - 다양한 컨테이너 런타임 지원 필요 => kubelet의 코드 수정 문제

      ![image](https://user-images.githubusercontent.com/38436013/106985274-7d59b900-67ac-11eb-9814-dd418e07ced5.png)

      

  - 3) CRI 등장

    - kubelet의 코드 수정x, 다양한 컨테이너 런타임 지원

      ![image](https://user-images.githubusercontent.com/38436013/106985291-83e83080-67ac-11eb-9639-ea067657987a.png)

### RKT

- 도커 외의 컨테이너, 로켓 컨테이너라고 부름
- 초기 버전 docker 보안 취약점 해결하기 위해 coreOS에서 탄생

### CRI-O

- OCI (open container initiative)
  - 컨테이너 종류 증가로 계속 CRI 구현해야 하는 문제 => 컨테이너 런타임 자체를 표준화
- 목적 : 레드햇이 만든 것으로, OCI 컨테이너 런타임으로 도커를 대체
- ONLY Kubernetes

- 도커의 한계 극복

  - 도커는 너무 무겁고, dockerEE는 구매해야 하는 단점

    

### 모놀리식 vs 마이크로서비스

![image](https://user-images.githubusercontent.com/38436013/106990112-f5c57780-67b6-11eb-91ef-857ac8a20136.png)

##### 모놀리식 아키텍처

- 장점
  - 어떤 기능이든지 개발된 환경 같아 단순 구조
  - end - to end 용이

- 단점
  - 한 프로젝트 덩치가 커져서 애플리케이션 구동시간, 빌드, 배포시간 길어짐
  - 조그마한 수정사항에도 전체 빌드, 배포
  - 부분 오류가 전체에 영향

##### 마이크로 서비스

- 장점
  - 기능별로 개발, 작업할당을 서비스 단위로 함
  - 수정 사항만 빠르게 빌드, 배포

- 단점

  - 관리 어려움, 서비스 분산으로 모니터링 어려움

  - 다른 서비스 호출하는 코드가 반드시 필요

  - end-to-end 테스트 위해 ui, gw 여러개의 마이크로서비스 구동해야함

    

### [Dockerfile 생성](https://docs.docker.com/engine/reference/builder/)

~~~
1. 디렉토리 만들기
   애플리케이션의 스크립트와 파일 디렉토리로 이동
   여기에 Dockerfile만들기
   
   Dockerfile 
   	- FROM 명령어, 컨테이너 기본이미지 선언
   	- 리소스 추가
   	- ADD, RUN, CMD
   	
2. 컨테이너 빌드
	> sudo docker build -t simpleapp
	
3. 이미지 확인
	> sudo docker 이미지
	> sudo docker run simpleapp
	
4. 저장소 푸시
	> sudo docker push
~~~

### 배포 생성

~~~
kubectl create 를 사용하여 이미지 테스트
> kubectl create deployment <Deploy-Name> --image = <repo> / < 앱이름> : <버전>
> kubectl create deployment time-date --image =  10.110.186.162 : 5000 / simpleapp : v2.2
~~~

### 컨테이너에서 명령 실행

~~~
kubectl exec -it <포드이름> -/bin/bash
~~~

### MULTI CONTAINER POD

- 파드의 모든 컨테이너는 단일 ip, 네임스페이스 공유

- 각 컨테이너는 파드에 제공된 스토리지에 대해 동일한 잠재적 액세스 권한

- *ambassador* , *adapter* , *sidecar*

- SIDECAR PATTERN

  - 하나의 컨테이너에는 하나의 책임만 가진다

  - 사이드카 패턴 : 서로 다른 역할을 하는 서비스는 각각의 컨테이너로 분리

  - 사이드카 컨테이너 : 메인 컨테이너를 확장하고 향샹시키며 개선하는 컨테이너

    <img src="https://user-images.githubusercontent.com/38436013/106992371-a2095d00-67bb-11eb-8954-a19b40aa5385.png" alt="image" style="zoom:40%;" />

  - 웹서버가 메인, 로그수집기가 사이드카 => 웹서버 바뀌어도 로그수집기 수정 안해도 됨

- AMBASSADOR PATTERN

  - 메인 컨테이너의 네트워크의 역할을 전담하는 프록시 컨테이너를 두는 패턴

    <img src="https://user-images.githubusercontent.com/38436013/106992545-075d4e00-67bc-11eb-86fd-61bea570ed71.png" alt="image" style="zoom:60%;" />

- ADAPTER PATTERN

  - 메인 컨테이너의 출력을 표준화

    <img src="https://user-images.githubusercontent.com/38436013/106992904-dd585b80-67bc-11eb-969f-ad1c91586d3c.png" alt="image" style="zoom:70%;" />

  - 모니터링 파드가 존재하고, 다른 파드의 모니터링 정보를 수집하는경우

  - 모니터링 파드에서 다른 파드의 CPU 사용량을 뽑고 싶다면, 메인컨테이너의 출력은 변화하지 않고, 어댑터 컨테이너만 수정해서 일관된 정보를 모니터링 파드에게 줄 수 있음

    

- 앰배새더, 어댑터 패턴을 사이드카 패턴으로 보지 않는 이유는?
  - 앰배새더, 어댑터는 메인 컨테이너로의 통신에 관여
  - 사이드카는 메인컨테이너의 기능을 확장하거나 개선시킨 것을 의미

### 준비 및 활성 프로브

- 프로브는 컨테이너에서 KUBELET에 의해 주기적으로 수행되는 진단

- readinessProbe
  - 컨테이너가 요청 처리 준비가 되었는지 여부
  - 실패 => endpoint 컨테이너는 연관된 모든 서비스들의 endpoint에서 파드의 ip 제거
  - 초기 지연 이전 기본 상태 : failure , 이프로브지원안하면 success
- livenessProbe
  - 컨테이너가 동작중인지 여부
  - 실패 => 컨테이너 죽이기 , 이 프로브 지원안하면 success

### 테스팅

- **$ kubectl describe pod test1**
  -   개체에 대한 세부 정보, 조건, 볼륨 및 이벤트를 볼 수있음
- **$ kubectl log test1**
  - 포드 내의 컨테이너 출력을 살펴 보는 것임
- **$ kubectl -n kube-system logs etcd-master**
  - 로그 출력
## 용어 정리
#### DOCKER , CONTAINER 비교

- 도커는 컨테이너 기술 중 하나
- 도커 =! 컨테이너

#### DOCKER

- 초기 도커 기술은 LXC기술 기반으로 구축, 현재 종속관계 벗어남

  ![image](https://user-images.githubusercontent.com/38436013/106986864-9c0d7f00-67af-11eb-83ca-f7e4cfabf534.png)

- LXC는 전체 애플리케이션을 하나로 실행,  도커는 개발 프로세스로 세분화
- 도커는 컨테이너 생성, 구축, 이미지 전송, 이미지 버전 관리 등 에 많은 사용자 호응
- 도커 이미지 파일은 각각 계층으로 이루어져 단일 이미지로 결합됨, 만약 이미지가 변경되면 계층이 생성되고, 사용자가 실행, 복사 명령을  지정할 때마다 새계층이 생성됨, 새로운 도커 컨테이너를 구축할 때 이전 계층 재사용으로 구축 빠름, 중간 변경 사항있다면 변경 내용이 공유되며 개선됨, 버전관리는 계층화와 관련이 있어 변경사항이 새롭게 발생할 때마다 내장 변경 로그가 기본적으로 적용됨

#### 컨테이너 오케스트레이션

- 대표적인게 kubernetes, docker swarm, apache mesos
- ''컨테이너 배포 관리'' 를 의미
- 목적 : 여러 컨테이너 관리, 컨테이너와 호스트 수가 많을 수록 가치 있는 기술, 이러한 유형의 자동화를 오케스트레이션이라고 함
- 이 중 쿠버네티스는 구글에서 설계, 클라우드 네이티브 컴퓨팅 재단에 기부됨

