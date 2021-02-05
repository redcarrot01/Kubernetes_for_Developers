# 02 KUBERNETES 아키텍처_1

### 목표

- Kubernetes와 관련된 가장 일반적인 용어 이해
- Kubernetes 역사에 대해 토론
- 마스터 노드 구성 요소
- 미니언 노드 구성 요소
- CNI(container network interface) 구성 및 네트워크 플러그인 이해

### kubernetes란?

- 컨테이너화된 애플리케이션의 배포, 확장 및 관리를 자동화하기 위한 오픈 소스 시스템
- Borg라는 프로젝트에서 구글에서 15년의 경험을 바탕으로 구축됨

### kubernetes의 구성 요소

- 대규모 서버 대신 다수의 소규모 웹 서버 또는 마이크로 서비스를 배포하는 방식

  ![image](https://user-images.githubusercontent.com/38436013/107011971-bd846000-67db-11eb-98b4-65204beb88b7.png)

- 컨트롤 플레인

  - 클러스터 제어 , 클러스터 상태 및 구성

  - kube-apiserver

    - 컨트롤 플레인의 프론트엔드
    - 내부 및 외부 요청 처리
    - etcd 디비에 연결하는 유일한 에이전트
    - rest 호출 , kubectl cli 통해 api에 접근

  - kube-scheduler

    - api 요청을 보고 해당 컨테이너를 실행하는데 적합한 노드 찾음

  - kube-controller-manager

    - 하나의 컨트롤러는 스케줄러를 참고하여 정확한 수의 포드가 실행, 파드 문제 감지 및 대응

  - etcd

    - 클러스터, 네트워킹 및 영구 정보 저장

    - 키 값 저장소 db

- 쿠버네티스 노드

  - 노드

    - 클러스터에서는 1개 이상의 노드 있음

  - 파드

    - 쿠버네티스에서 가장 작은 단위

    - 애플리케이션의 단일 인스턴스

    - 파드당 하나의 ip, 컨테이너가 두개 이상이라면 ip 공유

      ~~~
      $ kubectl run newpod --image = nginx --generator = run-pod / v1
      
      또는 올바른 형식의 JSON 또는 YAML 파일을 사용하여 생성 및 삭제
      
      $ kubectl create -f newpod.yaml
      
      $ kubectl delete -f newpod.yaml
      ~~~

  - kubelet
    - 워커 노드에서 컨트롤 플레인과 통신하는 것
    - 모든노드에 설치된 도커 엔진과 상호 작용
    - 컨테이너가 파드에서 실행되게 함
  - kube-proxy
    - 네트워크 연결
  - 퍼시스턴트 스토리지
    - 컨테이너, 데이터 관리
  - 컨테이너 레지스트리
    - 컨테이너 이미지 저장

- 레거시 vs 클라우드 아키텍처

  ![image](https://user-images.githubusercontent.com/38436013/107010100-3fbf5500-67d9-11eb-9e88-f6e345f882c6.png)



### 마이크로서비스 원칙

- 컨테이너 이미지 빌드, 테스트, 확인 => 지속적 통합 파이프라인 필요
- 컨테이너 실행할 머신 클러스터 
- 컨테이너 동작 감시하는 모니터링 시스템
- 롤링 업데이트 및 롤백
- 유연하고 확장 가능한 네트워크, 스토리지

### Kubernetes 아키텍처



![image](https://user-images.githubusercontent.com/38436013/107010644-f4597680-67d9-11eb-9c29-d4a3af2fd389.png)

- 마스터노드와 워커 노드로 구성
- 쿠버네티스는 api 서버를 통해 api 노출 -> kubectl이라는 로컬 클라이언트사용해 api와 통신 -> kube -scheduler는 api 요청 보고 컨테이너 실행 위한 노드 찾기 -> 각 노드는 kubelet, kube-proxy 두개 컨테이너 실행 -> kubelet 컨테이너는 컨테이너가 실행되거나 실패시 다시 시작되도록 함 ->  kube-proxy 컨테이너는 네트워크에 컨테이너를 노출

### MASTER 노드

-  클러스터에 대한 다양한 서버 및 관리자 프로세스를 실행

- 마스터 노드의 구성 요소 중에는 kube-apiserver, kube-scheduler 및 etcd 데이터베이스가 있음

### 서비스

- 파드의 집합에 정의하기 위한 경로를 정의
- 각 서비스는 여러 파드 간에 인바운드 요청을 분산하기 위한 단일 노드 포트 또는 로드밸런서와 같은 특정 트래픽을 처리하는 마이크로 서비스
- 리소스 제어 및 보안에 유용한 인바운드 요청에 대한 액세스 정책 처리
- 연결한 OBJECT를 알수 있는 셀렉터 2개
  - equality-based 레이블 키 및 해당 값을 기준
  - set-based 값 집합

### single IP per pod

- 포드는 일부 연결된 데이터 볼륨과 함께 배치 된 컨테이너 그룹
-  모든 컨테이너는 동일한 네트워크 네임 스페이스를 공유

- sudo docker ps

### network setup

- 파드는 같은ip 공유하는 컨테이너 그룹
- 파드는 vm 과 비슷
- 네트워크는 ip 주소를 파드에 할당, 모든 노드의 모든 파드 간에 트래픽 경로를 제공
- 컨테이너 조정 시스템에서 해결해야할 주요 네트워킹 과제
  - 컨테이너랑 컨테이너
  - 파드랑 파드
  - 외부랑 파드
- 파드들에게 애플리케이션 컨테이너가 시작되기 전에 ip 할당
- 서비스 개체는 네트워크 내에서 클러스터ip주소를 사용, 클러스터 외부에서 nodeport주소를 사용
- 로드밸런스 서비스로 구성된 경우 로드 밸런서를 사용하여 포드 연결하는데 사용

