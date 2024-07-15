### Kubernetes Security Primitives

kube api server -> 쿠버네티스의 모든 동작의 중심
api에 직접 접근하거나 kubectl을 통해 상호 작용하게 됨

해당 api 서버에 대한 접근을 어떻게 제어할 수 있을까? -> 1차 방어선이라고 할 수 있음

Authentication - who can access? -> 누가 kube-apiserver에 접근할 수 있는가

- Files-Username and passwords
- Files-Username and Tokens
- Certificates
- External Authentication providers - LDAP
- Service Accounts

Authorization - What can they do? -> 무엇을 할 수 있을까

- RBAC Authorization
- ABAC Authorization
- Node Authorization
- Webhook Mode

모든 component(ETCD, kube controller manager, api server, kubelet .. ) 사이의 통신 => TLS 암호화에 의해 보호
application 간의 통신은 => Network Policy에 의해 제한됨

---

쿠버네티스에 접근하는 Admin, Developers, Bots 들이 있다고 할 때,
이들에 대한 접근 제한이 필요로 함.

쿠버네티스는 기본적으로 사용자 계정을 관리하지 않고, 사용자를 관리하기 위한 파일, 인증서, LDAP같은 써드파티 서비스 등의 외부 서비스에 의존.

`그리고 모든 요청들은 대부분 kube-api server를 거치게 돼있음.`

Kubernetes 1.19에서 deprecated 된 방법이지만 .csv파일을 만들어서 kubeadm에 의해 구성된 kube-apiserver static pod를 편집하여 전달할 수 있음

---

TLS와 TLS trouble shooting은 어려울 수 있음

### TLS CERTIFICATES

- 대칭 키 방식은 키 자체가 노출될 수 있기에 안전한 방법이 아니다.
  - 네트워크로 결국 암호화에 사용한 키를 전달해야 하기 때문
- 비대칭 키 방법, public lock과 private key를 활용한 방식

그래서 대칭키를 옮기기 위해 비대칭키 암호화를 사용한다. 서버에 공용과 개인 키 쌍을 생성하는 것이다.

- openssl 명령어를 이용해 개인키와 공용키 쌍을 생성한다.

- 사용자가 https를 이용해 웹서버에 처음 접근하면 서버에서 공개 키를 받는다. 해커가 모든 네트워크를 엿보고 있으니 해커도 개인키와 공용키의 사본을 받았다고 가정하자.
- 사용자의 브라우저가 서버가 제공한 공용키를 이용해 대칭키를 암호화한다.
- 사용자는 공용키로 암호화된 대칭키를 서버로 보낸다. 해커도 사본을 받는다.
- 서버는 개인 키로 메시지를 해독하고 대칭키를 회수한다. 해커에겐 암호를 해독할 개인키가 없고, 공개키만 갖고 있기 때문에 메시지에서 대칭키를 되찾을 수 없다.
- 클라이언트와 서버는 대칭키로 데이터를 암호화해서 서로에게 보낼 수 있다. 상대방은 암호화된
  데이터를 대칭키를 통해 해독할 수 있다.

해커가 똑같은 서버를 만들어서 운영할 수도 있다.
그래서 서버가 키를 보내는 것이 아니라 인증서를 보내는 방법이 존재
인증서를 생성했으면 서명을 해야 하는데, 서명은 아무나 할 수 있다.

웹 브라우저가 신뢰할만한 인증서를 만드는데 역할을 누가 할까?
`CA(Certificate Authority)`

합법적인 CA가 서명했다는 것을 어떻게 알 수 있을까?

CA들은 개인 키와 공개 키 쌍을 가지고 있음. CA가 인증서를 서명할 때, 개인키를 사용함. 모든 CA의 공개 키는 브라우저에 기본으로 제공되기 때문에 브라우저는 CA의 공개 키를 활용하여 CA가 직접 서명한 인증서임을 확인함

결과적으로,

> 서버는 먼저 CA에 CSR(인증서 서명 요청)을 보낸다. CA는 개인키로 CSR에 서명한다. 모든 사용자는 CA의 공용키를 갖고 있다.
> 서명된 인증서는 서버로 다시 보내진다. 서버는 서명된 인증서로 웹 응용 프로그램을 구성한다.
> 사용자가 웹 응용 프로그램에 접근할 때마다 서버는 먼저 인증서(공개키)를 보낸다.
> 사용자, 정확히는 사용자의 브라우저가 인증서를 읽고 서버의 공개 키를 유효성 검사하고 회수하기 위해 CA의 공용키를 사용한다.
> 그런 다음 대칭키를 생성해 모든 통신에 사용하려고 한다. 대칭키는 서버의 공개키로 암호화되어 서버로 전송된다.
> 서버는 개인키로 메시지를 해독하고 대칭키를 회수한다.
> 대칭키는 앞으로 통신을 위해 사용된다.

### TLS in Kubernetes

kubernetes cluster 내에서도 통신하는데 있어 암호화가 필요함

- kubernetes-apiserver를 클라이언트들이 접근하기에 클라이언트를 위한 인증서들, 그리고 서버 내에서 kubelet과 같이 통신하는데 필요한 서버 인증서들이 존재함

1단계 - CA 구축
먼저 openssl genrsa 명령을 이용해 개인키 생성
openssl req 명령을 통해 인증서 서명 요청
인증서 서명 요청에 CN필드에 이름 지정
마지막으로 openssl x509 명령을 이용해 인증서에 서명

2단계 - 클라이언트 인증서 생성
openssl genrsa 개인키 생성
open req 명령을 이용해 인증서서명요청을 생성해 관리자의 이름을 지정
마지막으로, openssl xt509를 사용해 서명된 인증서를 생성.
CA 인증서와 CA 개인키를 지정 -> CA 개인키로 인증서에 서명. 이로써, 클러스터 내에 유효한 인증서가 생성됨

3단계 - 서버 인증서 생성
ETCD 서버의 인증서 생성 과정
ETCD 서버는 고가용성 환경에서 다중 서버에 걸쳐 클러스터에 배포될 수 있음. 이 경우, 클러스터 내 다른 멤버들 간의 통신을 안전하게 하려면 추가 피어 인증서를 생성해야 함

클러스터 내의 모든 작업은 kube-api를 거침. 클러스터 내 움직이는 모든 것을 kube-apiserver가 앎.

---

most certificates are stored in /etc/kubernetes/pki

### Certificates API

쿠버네티스 클러스터에서 마스터 노드에 CA 파일들이 위치하기 때문에ㅐ 마스터 노드는 CA서버

쿠버네티스에서 자동화된 방법으로 인증서를 관리하고 요청에 서명하는 Certificates API 존재

1. Certificates API를 통해 쿠버네티스로 인증서 서명 요청이 보내짐
2. 관리자가 인증서 서명 요청을 받으면 인증서 서명요청(CSR)을 생성
3. CSR 개체가 생성되고 나서 모든 인증서서명요청은 클러스터 관리자들이 볼 수 있음. kubectl 명령으로 요청을 검토 및 승인
4. 인증서는 추출되어 사용자와 공유

`인증서와 관련된 모든 작업은 controller manager에 의해 실행`

### KubeConfig

kubectl 도구는 기본값으로 `$HOME/.kube/config' 파일을 찾음

kubeconfig 파일에는 Clusters, Contexts, Users 3가지 영역이 존재

- Clusters는 접근이 필요한 다양한 쿠버네티스 클러스터를 말함
- Users는 클러스터에 접근 권한이 있는 사용자 계정 Admin, Dev User, Prod User 등
- Contexts는 사용자 계정이 어떤 클러스터에 접근하기 위해 사용되는지를 정의

### API Groups

쿠버네티스의 모든 리소스는 그룹화 되어 있음
상단 레벨에는 core API 그룹과 named API 그룹이 있음
named API 그룹 아래에는 다양한 리소스가 있고, 각 리소스엔 verbs라는 작업 모음이 있음

### ABAC과 RBAC

ABAC(Attribute-Based Access Control, 특성 기반 접근 제어)란 사용자나 사용자 그룹을 권한 모음으로 연결

RBAC는 사용자나 그룹을 권한 집합(policy)로 직접 연결하는 대신 역할을 정의

Role Based Access Controls
역할 기반 접근 제어

- Role 개체를 만들기 위해 Role 정의 파일을 작성
- Role 정의 파일에는 rules 영역이 있음. rules 영역은 apiGroups, resources, verbs 3가지 섹션이 존재

다음 단계는 사용자를 그 역할에 링크하는 것
Rolebinding 개체를 만들어서 사용자 개체를 Role에 연결

### Cluster Roles 와 Role Bindings

Role과 RoleBinding은 네임스페이스 내 생성

- 네임스페이스를 지정하지 않으면? default namespace에 생성

### Service Accounts

Service Account의 개념은 쿠버네티스의 다른 보안 관련 개념과 연결

서비스 계정은 컴퓨터가 사용하는 것
ex) Jenkins, Prometheus

### Network Policy

쿠버네티스 네임스페이스에 있는 또 다른 객체로 Pod, ReplicaSet, Service처럼 하나 이상의 파드에 네트워크 정책을 연결함

Ingress와 Egress

또한, 파드와 네임스페이스, ip주소를 지정하기 위해서 podSelector, namespaceSelector, ipBlock 영역을 사용하면 됨
