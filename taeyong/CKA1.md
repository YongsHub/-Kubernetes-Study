### Docker vs containerd

https://www.linkedin.com/pulse/containerd%EB%8A%94-%EB%AC%B4%EC%97%87%EC%9D%B4%EA%B3%A0-%EC%99%9C-%EC%A4%91%EC%9A%94%ED%95%A0%EA%B9%8C-sean-lee/?originalSubdomain=kr

### ETCD

ETCD는 신뢰할 수 있는 분산된 key-value 저장소 -> 간단하고 안전하고 빠른 저장소임

`Run ETCD Service`

```shell
./etcd
```

```shell
./etcdctl set key1 value1
```

```shell
./etcdctl get key1
```

ETCDCTL 버전 세팅을 통해 2 or 3로 변경할 수 있음

ETCD는 Nodes, PODS, Configs, Secrets, Accounts, ROles, Bindings, Others 등등 모든 클러스터 데이터에 대해 Back-Up 역할을 해줄 수 있음

kubectl get 명령을 실행할 때 표시되는 모든 정보는 ETCD 서버에서 가져온 것
노드 추가, 포드 또는 ReplicaSet, Deployment 등 클러스터에 대한 모든 변경 사항이 ETCD 서버에 업데이트됨

### Kube API Server

kube-api는 쿠버네티스에서 매우 중요한 management component이다

`kubectl 은 kube-api server에 접근할 수 있음`

kube-api server

1. Authenticate User
2. Validate Request
3. Retrieve Data
4. Update ETCD
5. Scheduler
6. Kubelet

특히, 스케쥴러나 Kubelet은 kube-api와 interact하면서 요청과 응답을 주고 받음

### Kube Controller Manager

쿠버네티스에서 다양한 컨트롤러들을 관리

- 상태 감시
- 상황 회복

### Kubelet

각 Node에서 Master Node와의 연결망

Kube-Scheduler -> kube-api sersver -> kubelet -> container engine
위와 같은 단계처럼 Pod를 생성하고, Node나 Pod 모니터링 역할 등을 수행함

Worker Node에 kubelet을 install해야 함

### Kube Proxy

클러스터의 각 노드에서 실행되는 프로세스

각 노드에 iptables 규칙을 이용해 트래픽이 향하도록 지원함

### YAML in Kubernetes

```yaml
//4개가 상위 Top level Property and Required Fields
apiVersion:
kind:
metadata:

spec:
```

### Practice Test - Pods

https://uklabs.kodekloud.com/topic/practice-test-pods-2/
