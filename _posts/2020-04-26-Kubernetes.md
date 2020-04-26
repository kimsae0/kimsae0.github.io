---
layout: post
title: Kubernetes
tags: [Cloud, Kubernetes, Pass, k8s, docker]
feature-img: "/assets/img/myimg/kubernetes.png"
thumbnail: "/assets/img/myimg/kubernetes.png"
image: "/assets/img/myimg/kubernetes.png"
author-id: kimsae0
excerpt_separator: <!--more-->
---

# Kubernetes 소개
![Kubernetes]({{ "/assets/img/myimg/kubernetes.png" | relative_url}})

<!--more-->

## Kubernetes란?
* Kubernetes는 Docker등과 같은 Container를 위한 Open Source Orchestration System
* 약 10여년 전, Google이 Container Management를 위해 개발한 Borg에서 시작, Omega를 거쳐 Kubernetes가 탄생
* 그리스 어로 배의 조타수를 의미(영어로는 Helmsman)
* Kubernetes 가운데 여덟글자를 생략하여 K8s라고 표기하기도 함
* 100% Open Source(Go언어로 개발) - https://github.com/kubernetes/kubernetes

## Kubernetes의 특징 
* Automatic bin packing
    * 가용성을 희생하지 않는 범위 내에서 리소스를 충분히 활용하여 Container를 배치

* Self-Healing
    * Container의 실행 실패, node가 죽거나 반응이 없는 경우, Health Check에 실패한 경우 해당 Container를 자동으로 복구

* Horizontal scaling
    * pod의 CPU 사용이나 App이 제공하는 Metic을 기반으로 ReplicaSet을 scaling 가능

* Service discovery and Load balancing
    * 익숙하지 않은 Service Discovery 매커니즘을 위해 Application을 수정할 필요가 없음\
    * Container에 고유 IP, 단일 DNS를 제공하고 이를 이용하여 Load Balancing

* Automated rollouts and rollbacks
    * Application, Configuration의 변경이 있을 경우, 전체 인스턴스의 중단이 아닌, 점진적으로 Container에 적용(Rolling update)\
    * 변경 내용에 문제가 있을 경우 자동으로 Rollback 진행

* Secret and configuration management
    * Secret Key, Configuration을 이미지 변경 없이 업데이트 및 외부 노출(expose)없이 관리/사용 가능

* Storage orchestration
    * Local storage를 비롯하여 Public Cloud(GCP, AWS, Softlayer 등), Network Storage등을 구미에 맞게 자동 Mount 가능

* Batch Execution
    * Batch, CI 작업의 수행 관리 가능 실패시 Container Replace 가능


## Kubernetes 아키텍처
* Kubernetes에서는 Kubernetes Master와 1개 이상의 Node(Worker)로 이루어진 가상/물리 머신의 Set을 통해 Cluster를 구성
![Travel]({{ "/assets/img/myimg/kube_archi.png" | relative_url}})

### Kubernetes Master
* Kubernetes Cluster의 Main Component
* __API Server(kube-apiserver)__
    * Kubernetes Components(kubectl, scheduler, replication controller, etcd datastore, kublet, kube-proxy)의 Hub로서 HTTP 및 HTTPS 기반의 RESTful API 제공
* __Scheduler(kube-scheduler)__
    * 어떤 Node에서 어떤 Pod가 실행될지 결정
    * CPU, Memory, 얼마나 많은 Container가 해당 Node에서 작동 중인가 등에 기반한 알고리즘을 통해 Container 배치
    * Resouce의 '공급'을 작업량의 '수요'와 일치시키는 것이 주 역할
* __Controller Manager(kube-controller-manager)__
    * Kubernetes node들을 관리
    * Kubernetes internal information을 생성하고 갱신
    * DaemonSet Controller 및 Replication Controller와 같은 핵심 Kubernetes Controller 실행
    * 관리하는 Resouces(Pod, Service Endpoint, etc) 등을 생성, 업데이트, 삭제하기 위해 API 서버와 통신
* __etcd__
    * 분산 Key/Value의 저장소
    * Cluster 설정 및 현재 상태(각 Node, Node에서 동작 중인 Pod등의 모든 상태) 저장


### Kubernetes Node(Minion)
* Kubernetes Cluster의 Slave(Work) node
* __Kublet__
    * Node마다 한 개씩 가지는 일종의 Node(Worker) Agent
    * Sevice, Pod를 실행/중지하고 Desired 상태를 유지시킴
    * 주기적으로 API Server에 Node의 상태를 Check&Report Access
* __Proxy(kube-proxy)__
    * Network Proxy + Load Balancer의 역할
    * 외부의 요청을 분산되어 있는 Pod으로 전달
* __Pod__
    * Kubernetes의 가장 작은 배포 단위
    * 1개의 Pod는 내부에 여러 개의 컨테이너를 가질 수 있으나 대부분 1~2개의 컨테이너를 가짐
    * 1개의 Pod는 여러개의 물리 서버에 나눠지는 것이 아니고 1개의 물리서버(Node)위에 올라 감
    * Pod 내부의 컨테이너들은 네트워크와 볼륨을 공유하기 때문에 Localhost로 통신 가능
    * Pod는 Cluster 내부에서 사용할 수 있는 유니크한 단독 네트워크 IP를 할당 받지만 해당 IP를 서비스에서 사용하지는 않음
    * Kubernetes에서 Pod는 언젠가 죽는(mortal) Object이기 때문
    * 동일 작업을 수행하는 Pod는 ReplicaSet이라는 Controller에 의해 정해진 Rule에 따라 복제 됨
    * 이 경우 복수의 Pod이 여러개의 Node에 걸쳐 실행 될 수 있음
* __cAdvisor__
    * 동작중인 Container의 Resource 사용량과 Performance를 모니터링
    * 모니터링 한 결과는 Kublet에 의해 API Server로 전달
* __Network Plugin__
    * Network Plugin은 다음과 같은 것들이 있음
    * Cluster Networking ACI, Cilium, Contiv, Contrail, Flannel, Google Compute Engine (GCE), Kube-router, L2 networks and linux bridging, Multus (a Multi Network plugin), NSX-T, Nuage Networks VCS (Virtualized Cloud Services), OpenVSwitch, OVN (Open Virtual Networking), Project Calico, Romana, Weave Net from Weaveworks, CNI-Genie from Huawei

### Object
* Kubernetes의 상태를 나타내는 Entity
* Kubernetes API의 Endpoint로서 동작
* 가장 기본적인 구성단위가 되는 Basic Object와 Basic Object를 생성하고 관리하는 등의 추가 기능을 가진 Controller로 이루어 짐
* 사용자가 기대하는 상태(Desired State)를 알 수 있는 Spec과 Desired State 대비 현재 상태를 나타내는 * Status 필드를 가짐
    * Object의 Status를 갱신하고 Spec에 정의 된 상태로 지속적으로 변화시키는 주체가 Controller
* __Object Spec__
    * Object의 특성(설정 정보)을 기술하여 Object를 정의
    * Command Line을 통해 Object 생성 시 인자로 전달하여 정의하거나 .yaml이나 .json으로 정의 가능
* __Basic Object__
    * Kubernetes에 의해 배포 및 관리되는 가장 기본적인 Object
    * Container화 되어 배포되는 Application의 Workload를 기술하는 Object
        * Pod
        * Service
        * Volume
        * Namespace
* __Pod__
    * Kubernetes에서 가장 기본적인 배포 단위Container를 포함하는 단위
    * Container를 개별적으로 하나씩 배포하는 것이 아니라 Pod라는 단위로 배포
    * Pod는 하나 이상의 Container를 포함Pod의 특징
    * Pod 내의 Container는 IP와 Port를 공유
    * Pod 내에 배포 된 Container 간에는 Disk Volume의 공유가 가능라우팅 가능 한 IP를 가짐
    * Kubernetes에서 Pod는 언젠가는 반드시 죽는(mortal) Object이기 때문
    * Service를 Pod에 연결했을 경우 Pod의 특정 포트가 외부로 노출됨
* __Service__
    * Pod들의 Persistent Endpoint
    * Service는 Pod 묶음의 Proxy로서 존재
    * Label과 Label Selector를 이용해 로드밸런서에서 Pod의 목록을 지정
    * 각 Pod에 지정 된 Label을 보고 서비스의 Label Selector에서 선택
* __Volume__
    * Pod이 기동될 때 Container에 기본적으로 생성되는 Local Disk는 영구적이지 못하기 때문에 Container의 재기동, 새로 배포할 시에 그 내용이 유실 됨
    * DB와 같이 영구적으로 저장해야하는 파일의 경우 Container의 재기동에 상관없이 영속적으로 저장 필요
    * 이 때 사용되는 것이 볼륨
    * 일종의 Container 용 외장 디스크
    * Pod가 기동 될 때 마운트해서 사용
    * Kubernetes의 볼륨은 Pod내 Container간에 공유 가능
* __Namespace__
    * Kubernetes Cluster 내의 논리적인 분리 단위
    * 물리적 분리가 아니기 때문에 다른 Namespace간의 Pod라도 서로간에 통신은 가능
    * Pod, Replicaset, Volume 등을 그룹핑 및 구분하는 데에 사용
    * 사용자별로 Namespace별 권한을 다르게 운영 가능
    * Namespace별로 Resource Quota(자원할당량) 지정 가능 및 Resource를 나눠서 관리 가능


### Controller
* Object를 Spec에 정의 된 상태로 지속적으로 변화시키는 주체
* Object를 생성 및 관리
    * Replication Controller
    * Replication Set
    * DaemonSet
    * Job
    * StatefulSet
    * Deployment
* __Replication Controller__
    * Pod를 관리해주는 역할
        * 지정 된 숫자로 Pod를 기동시키고 관리
    * Replica의 수, Pod Selector, Pod Template 3가지로 구성
        * Replica 수: Replication Controller에 의해서 관리되는 Pod의 수, 그 수만큼 Pod의 수를 유지
        * Selector: Label을 기반으로 Replication Controller가 관리하는 Pod를 가져오기 위해 사용
        * Pod Template: Pod를 추가로 기동할 경우 Pod 생성에 필요한 정보(Docker 이미지, Port, Label 등)를 정의
* __ReplicationSet__
    * Replication의 새 버전, 차세대 Replication Controller
        * Replication Controller는 점차 사용되지 않을 예정으로 기존의 Replication Controller의 역할은 ReplicationSet과 Deployment가 대신
    * Set 기반의 Selector 사용(Replication Controller는 Equality 기반 Selector 사용)
* __Deployment__
    * Replication Controller와 ReplicaSet의 좀 더 상위의 추상화 개념
    * 실제 운영에서는 앞선 두 Controller보다 더 많이 사용하게 됨
    * Application의 배포/삭제, Scale out의 역할
    * Deployment 생성 시 Deployment가 Pod와 ReplicaSets을 함께 생성
        * Pod에 Containerized Application들이 포함되고 Pod이 생성되면서 Application이 배포되는 원리
        * ReplicaSets은 Replica 수를 지속적으로 모니터링하고 유지 시켜 줌
* __DaemonSet__
    * Pod가 각각의 Node에서 하나씩만 돌게 하는 형태로 Pod을 관리하는 Controller
* __Job__
    * 원타임 작업이 필요한 경우처럼 Pod가 유지 될 필요가 없는 작업을 할 경우 사용되는 Controller
    * Job이 종료되면 Pod를 같이 종료
    * Container의 Spec에 Image 뿐만 아니라 Job을 수행하기 위한 Command를 같이 입력
    * Job이 비정상적으로 종료될 시 
        * 장애시 다시 시작 여부를 결정(Restart)
    * 같은 작업을 여러번 수행해야하는 경우 Pod를 순차적으로, 여러번 실행하도록 설정
        * completion 횟수를 Job 설정에 기입
    * 순차성이 필요 없고 병렬로 처리 가능한 일
        * parallelism에 동시에 실행할 수 있는 Pod의 수 기입
    * Cronjob
        * 정해진 스케쥴에 따라 Job Controller에 의해 작업을 실행해주는 Controller
* __StatefulSet__
    * Database와 같이 상태를 가지는 Pod을 지원하기 위한 Controller


### Service
* Pod들의 Persistent Endpoint 즉, Pod에 접근하는 방법을 기술하는 API
* Port나 Load Balancer에 대해 기술 할 수 있음
* Service는 IP 주소 할당방식 및 연동 서비스 등에 따라 크게 4가지로 구분
    * __Cluster IP__
        * Default 설정
        * Service에 Cluster IP(내부 IP)를 할당
        * Kubernetes 내부에서는 접근 가능하나 외부에서는 접근 불가
    * __Load Balancer__
        * 보통 Cloud Vendor에서 제공하는 설정 방식
        * 외부 IP를 가지고 있는 Load Balancer를 할당
        * 외부에서 접근 가능
    * __Node IP__
        * Cluster IP 뿐만 아니라 모든 Node의 IP와 Port를 통해서 접근 가능
    * __External name__
        * 외부 Service를 Kubernetes 내부에서 호출하고자 할 때 사용
* __Service Discovery__
    * Container가 자동으로 배치되기 때문에 어디에 배치되었는지를 찾는 임무를 수행 
    * Environment Variables, DNS 두 가지 방법으로 Service Discovery 수행

### Volume
* 상세 Volume Types(https://kubernetes.io/docs/concepts/storage/volumes/#types-of-volumes)
* Pod에 종속되는 디스크(Container 단위 X)
    * Pod에 속해 있는 여러개의 Container가 공유 가능
    * Volume의 종류
        * Temp
            * emptyDir
                * Pod가 생성될 때 생성되고 삭제될 때 같이 삭제되는 임시 Volume
        * Local
            * hostPath
                * Node의 Local Disk의 경로를 Pod에 마운트하여 사용
                * 같은 hostPath에 있는 Volume은 여러 Pod 사이에서 공유되어 사용됨
                * Pod가 삭제되더라도 hostPath에 있는 파일들은 삭제되지 않고 다른 Pod가 같은 hostPath를 마운트하게 되면, 파일에 액세스 가능
                    * Pod 재시작하여 다른 Node에서 재기동 될 경우 기존의 Node에서 사용한 hostPath의 파일에는 액세스 불가
        * Network
            * GlusterFS
            * gitRepo
            * NFS
            * iSCSI
            * gcePersistentDisk
            * AWS EBS
            * azureDisk
            * VshereVolume
