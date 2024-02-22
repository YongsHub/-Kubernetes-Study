## Manual Scheduling

Pod는 일반적으로 kube-scheduler에 의해 자동으로 어떤 Node에 배포될지 결정됨

하지만 스케줄러가 없으면? 엔지니어의 의도에 따라 배포하고 싶을 경우 어떻게 해야 할까?

### 스케줄러 동작 과정

1. 스케줄러는 모든 Pod을 보면서 nodeName 필드가 설정되어 있지 않은 Pod을 탐색
2. 스케줄링 알고리즘을 실행하여 Pod에 적합한 Node를 식별
3. 식별된 Node의 이름으로 nodeName 필드에 추가

### 스케줄러가 없다면?

- Pod은 계속해서 Pending 상태
- 엔지니어는 스케줄러 없이 매뉴얼하게 직접 Node에 Pod을 할당할 수 있다.
  이미 Pod가 생성되어 있고, Node에 할당하고 싶다면 Binding 객체를 생성 후 Binding API를 사용해서 POST요청을 보낼 수 있음

```
kubectl apply -f 파일이름
```

```
kubectl get pods -n kube-system
//node-scheduling이 없음
```

```
kubectl get pods --watch
```

```
kubectl replace --force -f 파일이름
```

## label and selector

쿠버네티스 클러스터에서 수많은 리소스가 있다면, 구분하기 위한 방법이 필요
즉, 어떤 리소스를 선택해서 명령을 실행하고자 할 때 Label과 Selector를 사용할 수 있음

### Label

- kubernetes Resource를 식별하기 위한 key-value 쌍의 메타정보
- 리소스의 하위 집합을 선택하고, 구성하는데 사용할 수 있음
- key는 고유한 값이어야 한다

### Selector

- Label을 이용해 쿠버네티스 리소스를 필터링하고 원하는 리소스 집합을 구할 수 있음
- `=`, `==`, `!=` 3가지 연산자만 허용

metadata 아래에 labels를 만들 수 있음

```
kubectl get pods --selector app=App1
```

```
kubectl get all --selector env=prod,tier=frontend,bu=finance
```

## Taints and Tolerations

Taint의 사전적인 내용을 보면 "오염"으로 풀이, 즉 Node가 오염되었기 때문에 Pod의 Schedule을 제한함
보통 Taint는 특정 Node에 대해 특정 Pod만 실행할 수 있도록 역할을 제한하기 위한 목적으로 많이 사용

Taint와 Toleration도 결국 Scheduling과 연관성을 가지고 있다

Taint는 node에 설정하고 Toleration은 Pod에 설정한다

Node에게 Pod를 받느냐 받지 않느냐에 대해 얘기하는 것이다

Master Node에 어느 Pod도 스케쥴되지 않는 것을 생각해본다면 Taint와 Tolerations를 떠올릴 수 있다

taint 방법

```
kubectl taint nodes node1 key1=value1:NoSchedule
```

untaint 방법

```
kubectl taint nodes node1 key1=value1:NoSchedule-
```

## Node Selectors & Node Affinity

결과적으로 Taint는 특정 node에 특정 values값을 가지는 pod만 배포하겠다는 의미이지. 해당 pod가 항상 특정 node에 배포되는 것을 보장하지는 못함

이런 문제점을 해결하기 위한 것이 NodeAffinity

NodeAffinity는 특정 pod를 특정 node에만 배포하는것을 보장함

Running중인 Deployment 수정은 kubectl edit을 활용할 수 있음

Edit Deployments
With Deployments you can easily edit any field/property of the POD template. Since the pod template is a child of the deployment specification, with every change the deployment will automatically delete and create a new pod with the new changes. So if you are asked to edit a property of a POD part of a deployment you may do that simply by running the command

Deployments의 경우, 자동으로 삭제하고 새로운 변화에 대한 새로운 파드를 생성해준다

## Resource Requirements and Limits

request -> 요청이라고 하는 것은 호스트에게 나에게 요청량만큼 확보해달라고 하는 것
limit -> 예를 들어, pod는 memory 1G를 요청했고 2G로 limit를 걸었을 때 최소한 memory 3G 중에서 1G는 확보해서 무조건 할당해주고 최대 2G까지 늘려서 쓸 수 있다는 것

## DaemonSet

EX) 모니터링 시스템 구축을 위해 모든 노드에 특정 파드를 관리해야 할 때 사용

```
kubectl create deployment elasticsearch -n kube-system --image=이미지이름 --dry-run=client -o yaml > fluend.yaml
```

```
kubectl get daemonsets -A
```

## Static Pods

cluster가 아니라 독립적인 Node만 존재하더라도 kubelet으로 Pod를 생성할 수 있음
kube-api에 의해 생성되는 pod도 아니고 directory에 yaml파일을 생성하여 pod를 생성할 수 있음

따라서 -> 이렇게 해야만 Pod를 생성할 수 있고 이것을 Static Pods라고 부름

### 특징

- 파드 이름에는 노드 호스트 이름 앞에 하이픈을 붙여 접미사로 추가된다.

--pod-manifest-path=로 경로를 지정할 수 있음 또는 config로 따로 설정할 수 있음

config file path 경로를 찾을때 cat /var/lib/kubelet/config.yaml 에서 확인할수 있다

## Multiple Scheduler

scheduleName으로 설정하거나 event log를 통해 어떤 schedular가 작업했는지도 확인할 수 있음
즉, 스케줄러는 하나만 존재할 수 있는 것이 아니라 여러 개 존재할 수 있음

### References

- https://dobby-isfree.tistory.com/166
- https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/?couponCode=KEEPLEARNING

- https://velog.io/@chancehee/posts
