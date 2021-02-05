## 1

![image](https://user-images.githubusercontent.com/38436013/106265941-1f3c4b80-626b-11eb-9d09-88b579c3597e.png)

## 2

- find : 어떤 파일이 어디있는지 찾고자 할때

- more : 파일 내용 확인, 파일을 화면단위로 끊어서 출력

  - 왼쪽 하단에 화면에 출력된 내용이 전체의 몇 % 인지를 표시하며, Enter 키를 입력하면 한 줄씩 출력되고, Space bar를 입력하면 한 화면씩 출력된다

  ![image](https://user-images.githubusercontent.com/38436013/106266974-637c1b80-626c-11eb-96a1-d82e0dd6b148.png)

  



- Run the script as an argument to the bash shell. You will need the kubeadm join command shown near the end of the output when you add the worker/minion node in a future step. Use the tee command to save the output of the script, in case you cannot scroll back to find the kubeadm join in the script output. Please note the following is one command and then its output.

- cp LFD259/SOLUTIONS/s_02/k8sMaster.sh .

- bash k8sMaster.sh | tee $HOME/master.out   

  - bash : 리눅스 표준 쉘, 명령 히스토리, 명령어 ㅗ안성 기능, 명령행 편집 ? 지원

  - shell : 사용자와 커널간의 다리역할, 사용자로부터 명령 받아 해석하고 프로그램 실행, 사용자가 로그인하면 각 쉘 부여되면서 명령어 수행 가능(쉘 부여 못밧으면 로그인해도 명령어 수행 못해)

    

- 미니노드

  - ![image](https://user-images.githubusercontent.com/38436013/106268687-c66eb200-626e-11eb-96ae-9de9caa879fc.png)

  - bash k8sSecond.sh 대신 
  - ![image](https://user-images.githubusercontent.com/38436013/106269081-5c0a4180-626f-11eb-9447-882d70d07a52.png)

  - When the script is done the minion node is ready to join the cluster. The kubeadm join statement can be found near the end of the kubeadm init output on the master node. It should also be in the file master.out as well. 
    Your nodes will use a different IP address and hashes than the example below. You’ll need to pre-pend sudo to run the script copied from the master node. Also note that some non-Linux operating systems and tools insert extra characters when multi-line samples are copied and pasted. Copying one line at a time solves this issue.

  - ㅇstudent@ckad-2:˜$ sudo kubeadm join --token 118c3e.83b49999dc5dc034 \
    10.128.0.3:6443 --discovery-token-ca-cert-hash \
    sha256:40aa946e3f53e38271bae24723866f56c86d77efb49aedeb8a70cc189bfe2e1d 에대한오류
    - ![image](https://user-images.githubusercontent.com/38436013/106269891-7c86cb80-6270-11eb-8817-1b5d9155d245.png)

- configure the master mode에서

  - 세컨 노드 오류 해결안했을때 마스터에서 확인한 결과

    ![image](https://user-images.githubusercontent.com/38436013/106270730-c15f3200-6271-11eb-9dca-78e9ed169439.png)
