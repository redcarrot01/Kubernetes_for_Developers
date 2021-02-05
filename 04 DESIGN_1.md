# 04 DESIGN

### 목표

- 응용 프로그램의 리소스 요구 사항 정의
- 멀티 컨테이너 포드 디자인 패턴 이해
- 응용 프로그램 디자인 개념

### DECOUPLED RESOURCE

-  인스턴스 수명 동안 각 구성 요소를 다른 리소스로부터 분리하는 것이 목표

- 서비스와 같은 중개자가 다른 리소스에 대한 연결 및 재연결 해줌

  

### 유연한 프레임 워크

- 여러개의 NGINX 서버를 배포, 각 서버는 워크로드의 작은 부분 처리
- kubernetes 오케스트레이션이 작동하려면 컨트롤러가 현재 클러스터 상태 지속적으로 모니터링, 해당 상태가 선언된 구성과 일치할 때까지 변경해야 함



### 리소스 사용 관리

- 스케줄링 : 쿠버네티스에서 pod를 어느 노드에 배포할지를 결정하는 것

- pod에 대한 스케줄링 시, 파드 내 애플리케이션이 동작할 수 있는 충분한 리소스(cpu, memory) 확보 필요

- 쿠버네티스는 필요한 리소스의 양, 리소스를 노드에 배포 

  

- .....

### 레이블 선택기 사용

- 레이블을 사용하면 다른 특성을 공유하지 않을 수 있는 개체 선택 가능

- 개발자가 자신의 이름을 사용하여 포드에 레이블을 지정하면 포드가 사용하는 애플리케이션 또는 배포에 관계없이 모든 포드에 영향을 미칠 수 있음

- 레이블은 운영자가 객체 추적, 관리하는 방법

- 선택기는 네임 스페이스 범위,  **--all-namespaces** 인수를 사용하여 모든 네임 스페이스에서 일치하는 객체를 선택

  ~~~
  $ kubectl get <obj-name> -o yaml 
  
  예를 들면 :
  
  ckad1 $ kubectl get pod examplepod-1234-vtlzd -o yaml
  
  apiVersion : v1
  종류 : 포드
  메타 데이터 :  
    주석 :     
      cni.projectcalico.org/podIP : 192.168.113.220/32   
    creationTimestamp : "2020-01-31T15 : 13 : 08Z"   
    generateName : examplepod-1234-   
    라벨 :     
      앱 : examplepod     
      pod-template -해시 : 1234   
    이름 : examplepod-1234-vtlzd
  ....
  
  이 출력을 사용하여 포드를 선택할 수있는 한 가지 방법은 앱 포드 레이블 을 사용하는 것 입니다. 명령 줄에서 콜론 (:)은 등호 (=) 기호로 대체되고 공백은 제거됩니다. 의 사용 -l 또는 --selector 옵션을 사용할 수 있습니다 kubectl .
  
  ckad1 $ kubectl -n test2 get --selector app = examplepod pod
  
  이름 준비 상태 다시 시작 나이
  examplepod-1234-vtlzd 1/1 달리기 0 25m
  
  몇 가지 기본 제공 개체 레이블이 있습니다. 예를 들어 노드에는 특정 노드 또는 노드 유형에 포드를 할당하는 데 사용할 수있는 arch , hostname 및 os 와 같은 레이블 이 있습니다.
  
  ckad1 $ kubectl 노드 작업자 가져 오기
  
  ....
     creationTimestamp : '2020-05-12T14 : 23 : 04Z'
     라벨 :
       beta.kubernetes.io/arch : amd64
       beta.kubernetes.io/os : linux
       kubernetes.io/arch : amd64
       kubernetes.io/hostname : 작업자
       kubernetes.io/os : linux
     managedFields :
  ....
  
  podspec 의 nodeSelector : 항목은이 레이블을 사용하여 다음과 같은 항목이있는 특정 노드에 포드를 배포 할 수 있습니다.
       spec :
         nodeSelector :
           kubernetes.io/hostname : 작업자
         컨테이너 :
  ~~~

  

