# Fluent Bit를 DaemonSet으로 설정하여 CloudWatch Logs에 로그 전송하기
[공식문서](https://docs.aws.amazon.com/ko_kr/AmazonCloudWatch/latest/monitoring/Container-Insights-setup-logs-FluentBit.html)


### Fluent Bit란?
* Fluentd의 경량화 버전

<br>

### 적용방법
1. Cloud Watch IAM 정책을 EKS role에 추가
2. eks cluster에 namespace 생성
```SHELL
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/cloudwatch-namespace.yaml
```
3. ConfigMap 생성
```SHELL
ClusterName=cluster-name
RegionName=cluster-region
FluentBitHttpPort='2020'
FluentBitReadFromHead='Off'
[[ ${FluentBitReadFromHead} = 'On' ]] && FluentBitReadFromTail='Off'|| FluentBitReadFromTail='On'
[[ -z ${FluentBitHttpPort} ]] && FluentBitHttpServer='Off' || FluentBitHttpServer='On'
kubectl create configmap fluent-bit-cluster-info \
--from-literal=cluster.name=${ClusterName} \
--from-literal=http.server=${FluentBitHttpServer} \
--from-literal=http.port=${FluentBitHttpPort} \
--from-literal=read.head=${FluentBitReadFromHead} \
--from-literal=read.tail=${FluentBitReadFromTail} \
--from-literal=logs.region=${RegionName} -n amazon-cloudwatch
```
4. Fluent Bit DaemonSet 다운로드 및 배포
```SHELL
# 최적화 구성을 사용함
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/fluent-bit/fluent-bit.yaml
```
5. 확인
```SHELL
kubectl get pods -n amazon-cloudwatch
```
6. 설정확인
* CloudWatch > 로그 그룹 > 검색 > /aws/containerinsights/Cluster_Name 경로 하위에 application, host, dataplane 경로가 있어야함


<br>

### 추가 과제
* 어떤 데이터를 수집해야 할까?
![image](https://github.com/user-attachments/assets/7110e6a7-bc7d-42b2-9c8a-774e895e5c7f)

* Controll Plane 로깅 설정 On EKS
    * EKS > 클러스터 이름 > 관찰성 > 컨트롤 플레인 로깅 섹션 > 로깅 관리 > 필요사항 설정 > 저장


<br>


###참고
* Grafana에 대한 기술 지식 습득이 되기 전 로그를 보기위해 임시로 붙임. 사장 되는 기술스택