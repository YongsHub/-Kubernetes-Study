### Ddcker Storage

- Storage Drivers
  Docker - Storage Drivers: AUFS 리눅스 파일 시스템을 기반 Layered Architecture

- Volume Drivers
  Volume은 Storage Drivers가 관여하지 않음. `Default Volume Plugin은 Local이다` 해당 Local Volume Plugin이 볼륨 생성을 도우고 데이터를 var/lib/docker/volumes 디렉토리에 생성해줌

### Container Storage Interface (CSI)

Kubernetes에 다른 컨테이너 런타임이 들어오면서 확장 및 통신을 위해 Interface가 필요해짐

따라서 컨테이너 런타임은 CRI 표준만 잘 따르면서 개발할 수 있음

### Container Network Interface (CNI)

네트워킹 솔루션 확장

### Container Storage Interface(CSI)

다중 저장소 솔루션을 위해 개발됨
CSI는 컨테이너 오케스트레이터가 호출할 일련의 RPC 또는 원격 프로시저 호출을 정의하며 이러한 호출은 스토리지 드라이버에 의해 구현되어야 함
`예를 들어, CSI는 포드가 생성되고 볼륨이 필요한 경우 컨테이너 오케스트레이터(쿠버네티스)가 볼륨 생성 RPC를 호출하여 볼륨 이름과 같은 세부 정보 집합을 전달`

스토리지 드라이버는 이 RPC를 구현하고 해당 요청을 처리함

### Volume

DOcker Volume과 마찬가지로, Pod 또한 데이터가 일시적이기 때문에 Volume이 필요함

단, 단일 노드일 경우 쿠버네티스가 Local에 볼륨 저장소를 생성하여 보관할 수 있으나 다중 노드일 경우 노드에 볼륨을 설정하는건 적합하지 않음

왜냐하면, 서로 다른 서버에 있기 때문에 `외부 복제된 스토리지 솔루션을 구성해야 함`

### Persistent Volumes

유저는 항상 Pod를 생성하거나 할 때, Volume을 마운트하고 직접 연결해야 함. 이럴 때, PV Pool로 관리되는 것이 편리함

이렇게 PV가 Pool을 가지고 이를 PVC(Persistent Volume Claims)를 통해서 얼마나 사용 할지 또는 어디에 연결할 지 등을 정해서 사용함

PV, PVC는 1:1 관계로 사용 가능함

### Persistent Volumee Claim

PV Pool을 PVC로 나눠서 바인딩을 해주는 개념으로 생각하기

`PVC와 PV 동적으로 바인딩`

### Storage Class

정적 프로비저닝 볼륨은 수동으로 해야하니 불편한 점이 있음

application의 volume을 사용하기 위해서는 gcloud를 써서 정적인 프로비저닝을 해야함. 이는 매우 자주 있는 일이니 자동화하는게 좋음
이를 자동화 한 것이 `동적 프로비저닝`

여기서는 google cloud 프로비저너만 예를 들었지만 AWS의 EBS, Azure File 등과 같은 다양한 종류의 프로비저너가 존재함
