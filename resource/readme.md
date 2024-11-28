http://containerlab24.com/metric/metric.html
- Metric-server deploy
```sh
[vagrant@master ~]$ kubectl top nodes
error: Metrics API not available   <= Metric 안됨.
```

github:
https://github.com/kubernetes-sigs/metrics-server
[vagrant@master ~]$

installation:
```sh
# 설치
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
# 삭제
kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

실행했을때 에러가 나면 아래처럼 파일을 다운로드 받은다음에 수정해서 다시 실행하여야 한다.

난 되는데..? 
안되는군...ㅋ

* 기존의 리소스는 삭제
=> 배포한것을 삭제하려면 => kubectl delete -f componets.yaml

[vagrant@master ~]$ vim componets.yaml
....
....
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=xxxx
        - --kubelet-insecure-tls  <== 한줄추가.
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        image: k8s.gcr.io/metrics-server/metrics-server:v0.5.2
...

[vagrant@master ~]$ kubectl apply -f components.yaml


[vagrant@master work]$ kubectl get pods -n kube-system  ; 아래처럼 1/1 Running 인지 확인
<= 시간이 1~2분 정도 걸릴수 있습니다.

NAME                                         READY   STATUS    RESTARTS       AG                                         E
....
metrics-server-7795cfc6ff-n4rb5              1/1     Running      0              63                                         s
...
...

[vagrant@master work]$ kubectl top nodes
NAME                 CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
master.example.com   232m         11%    1242Mi          33%
node1.example.com    63m          3%     450Mi           12%
node2.example.com    46m          2%     401Mi           10%

1000milicore => 1cpu => 1 core => 1 AWS vCPU , 1GCP Core  
100milicore => 0.1cpu => 0.1 core => 0.1 AWS vCPU , 0.1 GCP Core.