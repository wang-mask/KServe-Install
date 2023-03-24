# 该 Demo 用于在国内环境下安装 KServe

## 环境准备
需要提前安装:
- Helm
- K8s
- kubelet

## 版本
本脚本中已经固定了以下组件的脚本：
- istio: 1.15.0
- knative: v1.7.0
- cert manager: v1.9.0
- KServe: v0.10.1

由于国内 grc 镜像仓库问题，无法安装 KNative，该项目中的部署文件已经替换为 Docker Hub 镜像。此外，由于 KServe 官方的 Yaml 部署文件问题，会导致在 K8s 1.22 以上版本部署时出现控制器 CrashLoopBackOff 问题，故本项目采用 Helm 安装。

如果需要更换组件脚本，请替换对应文件夹下的 Yaml 文件或者 Chart 包。

## 安装
    chmod +x start.sh && ./start.sh

## 测试
```shell
# apply the demo
kubectl apply -f demo/

# wait demo ready
kubectl get inferenceservices sklearn-iris -n kserve-test
NAME           URL                                                 READY   PREV   LATEST   PREVROLLEDOUTREVISION   LATESTREADYREVISION                    AGE
sklearn-iris   http://sklearn-iris.kserve-test.example.com         True           100                              sklearn-iris-predictor-default-47q2g   7d23h
```
不知道为何并没有创建对应的 Service，所以无法通过对应的 service 访问模型服务，故直接访问了模型 POD 对应的 IP。
```shell
# apply the demo
kubectl apply -f demo/

# wait demo ready
kubectl get inferenceservices sklearn-iris -n kserve-test
NAME           URL                                                 READY   PREV   LATEST   PREVROLLEDOUTREVISION   LATESTREADYREVISION                    AGE
sklearn-iris   http://sklearn-iris.kserve-test.example.com         True           100                              sklearn-iris-predictor-default-47q2g   7d23h
```
```shell
# get the pod ip
export HOST=$(kubectl get pods -n kserve-test   -o jsonpath='{.items[0].status.podIP}')

# get the resault
curl -v "http://${HOST}:8080/v1/models/sklearn-iris:predict" -d @./iris-input.json
* About to connect() to 100.127.122.197 port 8080 (#0)
*   Trying 100.127.122.197...
* Connected to 100.127.122.197 (100.127.122.197) port 8080 (#0)
> POST /v1/models/sklearn-iris:predict HTTP/1.1
> User-Agent: curl/7.29.0
> Host: 100.127.122.197:8080
> Accept: */*
> Content-Length: 76
> Content-Type: application/x-www-form-urlencoded
> 
* upload completely sent off: 76 out of 76 bytes
< HTTP/1.1 200 OK
< date: Fri, 24 Mar 2023 07:31:58 GMT
< server: uvicorn
< content-length: 21
< content-type: application/json
< 
* Connection #0 to host 100.127.122.197 left intact
{"predictions":[1,1]}#                   
```