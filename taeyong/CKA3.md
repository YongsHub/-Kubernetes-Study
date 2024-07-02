## Monitor Cluster Components

- Node Level의 모니터링
- Pod Level의 모니터링

이런 메트릭을 모니터링하고 저장하고 데이터에 대한 분석을 제공할 수 있는 솔루션이 필요함

METRICS SERVER는 클러스터에 대해 1개가 필요하고 각각의 노드와 파드로부터 데이터를 가져와서 메모리에 저장함
IN-MEMORY에 저장

```shell
kubectl top node

kubectl top pod
```
