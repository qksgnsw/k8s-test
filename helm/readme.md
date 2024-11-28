helm

https://helm.sh/ko/docs/intro/install/

```sh
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version
```

https://artifacthub.io

example
https://artifacthub.io/packages/helm/nicholaswilde/mariadb


bitnami
helm repo add bitnami https://charts.bitnami.com/bitnami
helm pull bitnami/nginx
tar -xf nginx-18.2.6.tgz
helm install nginx -f my-values.yaml .


프로메테우스
https://artifacthub.io/packages/helm/prometheus-community/prometheus

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm pull prometheus-community/kube-prometheus-stack
tar xvfz kube-prometheus-stack-66.3.0.tgz
mv kube-prometheus-stack kube-prometheus-stack-custom
cd kube-prometheus-stack-custom/
cp values.yaml my-values.yaml # 책 p304-305 참조. 그라파나 템플릿에서 storage class name은 변경하지 않음
helm install prometheus -f my-values.yaml .

loadbalancer의 포트 확인하여 접속하면 된다. 오 신기
근데 지금 metallb 안깔려 있어서 펜딩뜸. 그래도 접속은 됨

확인을 위해 임시로 포트포워딩해보자
내부에서 호출시
kubectl port-forward svc/prometheus-prometheus-node-exporter 8080:9100
Forwarding from 127.0.0.1:8080 -> 9100
Handling connection for 8080

외부에서 호출시
kubectl port-forward --address=127.0.0.1,192.168.31.10 svc/prometheus-prometheus-node-exporter 8080:9100
Forwarding from 127.0.0.1:8080 -> 9100
Forwarding from 192.168.31.10:8080 -> 9100
Handling connection for 8080

프로메테우스 ui로 접근하기.
kubectl port-forward --address=127.0.0.1,192.168.31.10 svc/prometheus-kube-prometheus-prometheus 8080:9090
Forwarding from 127.0.0.1:8080 -> 9090
Forwarding from 192.168.31.10:8080 -> 9090
Handling connection for 8080
Handling connection for 8080
Handling connection for 8080


그라파나 초기 비밀번호 알아내기
kubectl get secrets prometheus-grafana -o yaml
apiVersion: v1
data:
  admin-password: cHJvbS1vcGVyYXRvcg==
  admin-user: YWRtaW4=
  ldap-toml: ""
kind: Secret
metadata:
  annotations:
    meta.helm.sh/release-name: prometheus
    meta.helm.sh/release-namespace: monitoring
  creationTimestamp: "2024-11-28T06:53:34Z"
  labels:
    app.kubernetes.io/instance: prometheus
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: grafana
    app.kubernetes.io/version: 11.3.1
    helm.sh/chart: grafana-8.6.3
  name: prometheus-grafana
  namespace: monitoring
  resourceVersion: "16417"
  uid: 744c334a-3608-4ee2-80ec-22215f9fb5b7
type: Opaque

echo cHJvbS1vcGVyYXRvcg== | base64 -d
prom-operator
echo YWRtaW4= | base64 -d
admin