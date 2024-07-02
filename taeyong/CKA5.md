### Rollout And Versioning

- 처음 배포를 생성하면, 롤아웃을 Trigger 함. 그리고 새로운 롤 아웃은 리비젼1을 생성함. 따라서, kubectl을 통해 이전 history들을 체크할 수 있음

### Deployment Strategy

- 새로운 버전을 위해, 현재 배포된 인스턴스들을 Down 후, 새로운 인스턴스를 실행하는 방법이 있음.

이 방법의 문제는 Application Down된 시간 동안 사용자가 접근이 불가능하다는 특징이 존재
이러한 방법을 `Recreate 전략`이라고 부름. 다행이도 기본 전략은 아니다!

- Rolling Update: 하나씩 구 버전을 Down하고 새로운 버전을 생성하며 단계적으로 수행

* 새롭게 배포된 Replica Set에 문제가 있다면? 이전 버전을 통해 Rollback할 수 있음 예를 들어,

```shell
kubectl rollout undo deployment/myapp-deployment
```

deployment image update
container name: simple-webapp

```shell
kubectl set image deployment/frontend simple-webapp=kodekloud/webapp-color:v2
```

### Configure Applications

- 실행하는 컨테이너의 Default Command를 정의 내릴 수 있음 Docker command 참조

예를 들어,

```Dockerfile
FROM Ubuntu

CMD sleep 5
```

ENTRYPOINT를 통해, Default 명령을 지정할 수 있다

```Dockerfile
FROM Ubuntu

ENTRYPOINT ["sleep"]

CMD ["5"] // 기본 값
```

만약, ENTRYPOINT를 재정의하고 싶으면 --entrypoint로 지정할 수 있음

![cmd](./picture/docker&kubernetes%20cmdAndArgs.png)

pod를 정의하는 definition.yml에서 ENTRYPOINT에 대한 재정의는 command를 통해 재정의하는 것이다.
CMD는 args를 통해 재정의함

### ENV Variables in Kubernetes

우리가

```shell
docker run -e APP_COLOR=pink simple-webapp-color
```

-e 옵션으로 환경 변수를 설정할 수 있었는데,

pod생성을 위한 yml에서는

```yml
env:
  - name: APP_COLOR
    value: pink
```

로 정의할 수 있음

이외에도 `valueFrom: conf` 와 `valueFrom` 이 있다.

- ConfigMap을 활용하는 방법: 1. Create ConfigMap 2. Inject into Pod
- secret을 활용하는 방법. 1. Create secret 2. Inject into Pod

`secret 생성 시, 주의 사항`
시크릿 값은 인코딩된 형식으로 제공해주는게 안전함
base64 encoding

- pod에서 secret을 주입하기 위해서는 envFrom secretRef name:에 정의해주면 된다

### Secret 주의 사항

- Secret은 암호화되어 있지 않음. 오직 인코딩 되어 있음
- kubernetes docs에 암호화에 대한 내용이 있으니 참고
- Secrets들을 암호화할 수 있는데 ETCD에 저장됨

### Monolith 에서 MicroServices

- 재사용 가능한 코드 세트를 개발 및 배포할 수 있음
- Scale Up And Down

예를 들어, Web Server와 Log Agent를 하나의 Network에 함께 구성할 수 있음
즉, 하나의 Pod에 Multi Container를 허용하기 때문이다

하나의 POD에서 multi-container를 구성하기 위한 design pattern이 존재

1. sidecar pattern
2. adapter pattern
3. ambassador pattern

### Init Container

멀티 컨테이너 포드에서는 각 컨테이너가 포드의 라이프사이클 동안 계속 실행되어야 함 예를 들어, 웹 애플리케이션과 로깅 에이전트를 포함한 멀티 컨테이너 포드에서는 두 컨테이너 모두 항상 살아 있어야 함. 로깅 에이전트 컨테이너에서 실행되는 프로세스는 웹 애플리케이션이 실행되는 동안 계속 살아 있어야 함 -> `이들 중 하나라도 실패하면 포드는 재시작`

하지만 때때로 포드가 처음 생성될 때 `한 번만 실행되는 프로세스를 실행하고자 할 수 있음` 예를 들어, 메인 웹 애플리케이션에서 사용할 코드나 바이너리를 저장소에서 가져오는 작업은 포드가 처음 생성될 때 한 번만 실행하면 됩니다. 또는 실제 애플리케이션이 시작되기 전에 외부 서비스나 데이터베이스가 준비될 때까지 기다리는 프로세스도 있을 수 있습니다. 이럴 때 `initContainers를 사용`

`initContainer는 다른 컨테이너와 마찬가지로 포드에 구성되지만, initContainers 섹션 내에 지정`

### Self Healing Applications

Kubernetes는 ReplicaSets와 Replica Controllers를 통해 셀프 힐링을 지원함

특히, Replica Controller는 애플리케이션 내 POD crash가 생겼을 때, 자동으로 re-created해줌

또한, Kubernetes는 실행중인 PODS내 health check를 위한 지원을 제공함
`Liveness 와 Probes를 통해 지원`
