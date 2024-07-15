Node가 하나 사라졌을 경우, ReplicaSet으로 관리되던 Pod는 다시 다른 노드로 롤링 업데이트가 될 것임

이때, 노드가 내려갔다가 다시 올라 옴 (uncordon) 그러면 기존에 붙어 있던 Pod가 해당 Node로 붙지 않음.

만약, OS Parameter를 바꾸게 되어서 Node 하나를 재부팅 해야 하면 Node Drain을 진행해야 함

- 이때, drain 명령어를 사용하면 기존 노드에 존재하던 노드의 포드가 옮겨지고 종료됨

```shell
kubectl drain node-1
```

하지만, ReplicaSet 없이 Single Pod가 붙어 있다면 강제 drain을 시켜야 드레인이 가능함. --force

cordon 명령어는 기존 POD는 그대로 두고 해당 노드의 스케줄링만 중단시킨다.

```shell
kubectl cordon node-1
```

이후, 아래 명령어로 노드 스케줄링을 허용해서 다시 본인의 노드로 새로운 POD 배치를 가능케 함

```shell
kubectl uncordon node-1
```

---

### Kubernetes Software Versions

v1.11.3

1. 1: Major
2. 11: Features, Functionalities
3. 3: Bug Fixes

이외에도 알파와 베타로 나뉠 수 있음.
모든 버그를 고치고 개선시키면, 먼저 알파 태그를 부착한 릴리즈가 이루어지고 다음으로 베타 릴리즈가 출시됨 이후, 안정적으로 출시함

### Cluster Upgrade Process

쿠버네티스 구성 요소로 kube-apiserver, controller manager, kube-scheduler, kubelet, kube-proxy, kubectl, ETCD CLUSTER, CoreDNS 등이 있음

하지만 이중에서도 우리는 핵심 코어에 집중함. 그런데 각각의 구성 요소가 같은 버전을 가져야만 할까?

`구성 요소는 다른 버전으로 갈 수 있음`

하지만 그 누구도 kube-apiserver보다 높은 버전을 가질 수 없음

Master Upgrade

- d

Worker Upgrade

- 한꺼번에 업그레이드
- 한번에 노드 하나씩 업그레이드
- 클러스터에 새 노드를 추가하고 Pod를 옮기고 Old Node를 삭제

GKE 같은 경우에는 클릭만으로 업그레이드가 가능함.

kubeadm은 upgarde plan을 가지고 있음
Master Node가 먼저 업그레이드 되고, 그 다음에 Worker Node가 업그레이드 됨

### Backup and Restore

- kubeapi-server를 활용한 백업
  kubeAPIServer를 이용해서 쿼리를 하는 것인데 API에 직접 액세스 하고 이를 카피로 두기 때문에 누락된 객체에 대한 걱정을 할 필요 없음

```shell
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

- ETCD Cluster를 활용한 백업
  ETCD는 클러스터의 상태를 공유하고 저장함.
  ETCD는 Master Node에 hosted되어 있어서 어디에 데이터가 저장될 지 지정할 수 있음

또한, Snapshot Solution을 가져오는데 빌트인 되어 있음

ETCD Control Utility를 사용해서 정상적인 상태의 클러스터를 스냅샷을 찍을 수 있음
