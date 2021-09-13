# kubernetes

## Install packages
- poetry add awscli
- Переходим на сайт https://aws.amazon.com/ru/eks/
  

### Устанавливаем eksctl (https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)
- curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
- sudo mv /tmp/eksctl /usr/local/bin
- eksctl version

### Устанавливаем kubectl (https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
- curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
- chmod +x kubectl 
- sudo mv ./kubectl /usr/local/bin
- kubectl version

### Установка ключей от AWS аккаунта:
- export AWS_ACCESS_KEY_ID=XXXXXXXXXXXXXXXXXXXXXXX
- export AWS_SECRET_ACCESS_KEY=XXXXXXXXXXXXXXXXXXXXXXX
- export AWS_DEFAULT_REGION=us-east-2

### Запуск кластера по умолчанию через eksctl
- eksctl create cluster --name test-k8s
- kubectl cluster-info
- kubectl get nodes
- eksctl delete cluster --name ilgizar-k8s

#### Полезная информация по eksctl:
- Документация по eksctl -> https://eksctl.io/introduction/
- Примеры конфигов по eksctl -> https://github.com/weaveworks/eksctl/tree/main/examples

##### Что такое eksctl ?
eksctl - расшифровывается как Amazon Elastic Kubernetes Service. Он представляет собой CLI (command-line interface) командную строку для управления кластером в AWS. Написан на Go.
Для eksctl можно написать файл конфигурации и использовать его для разворачивания кластера в AWS.

##### Запуск кластера через конфиги
- eksctl create cluster -f mycluster.yaml
- eksctl delete cluster -f mycluster.yaml


## История
- Зарегался в AWS и создал аккаунт
- Установил утилиты eksctl, kubectl, aws-cli
- Перешел на сервис EKS и создал кластер
- Посмотрел EC2 instance - были созданы 2 ноды
- Удалил кластер
- Создал файл конфигов для eksctl - mycluster.yaml
- Запустил кластер через eksctl через конфиг - были созданы ноды с указанными серверами
- Остановил кластер

### EC2
- Перешел в вкладку EC2
- Открыл страницу Instances
- Нажал на Launch Instanses
- Выбрал Amazon Linux 2 AMI (HVM), SSD Volume Type (64-bit (x86)) и нажал Select
- Выбрал обращ t2 `t2.micro` (Free tire eligible)
- Нажал `Review and Launch`
- Затем нажал `Launch`
- Из выпадающего списка выбрал `Create a new key pair`
- Назвал online-course
- Нажал `Download Key Pair` и скачал файл в /home
- В итоге создается сервер
- Открываю `EC2 instances` и вижу свой арендованный сервер
- Из выпадающего списка выбираю свой созданный сервер
- Нажимаю `Connect`
- Выходит таблица с 4-мя разделами
- Перехожу в раздел `SSH client`
- Выполняю команды из описания для подключения к серверу
- Открыл терминал
- Перешел в директорию `/home`
- Выполнил команду `chmod 400 online-course.pem`
- Затем подключился к серверу `ssh -i "online-course.pem" ec2-user@ec2-3-68-71-93.eu-central-1.compute.amazonaws.com`
- Выполнил все команды из описания https://gist.github.com/npearce/6f3c7826c7499587f00957fee62f8ee9
- Перезагрузил сервер `sudo reboot 0`
- Подключился повторно к серверу через `ssh`
- Загрузил тестовый проект https://github.com/tbublik/example-web-app-for-amazon-ec2
- `git clone https://github.com/tbublik/example-web-app-for-amazon-ec2`
- `docker-compose up --build`
- В EKS Instance выделяем наш сервер и смотрим описание
- Открываем в браузере ссылку https://{ipv4 public}}/ (`http://3.68.71.93/`) - не открывается
- Переходим в EKS в Instance в раздел `Security` и кликаем на `Security groups` - открывается новая страница
- Выделяем нашу группу и открываем раздел `Inbound rules`
- Нажимаем `Edit inbound rules` -> `Add rule`
- Добавляем Type `HTTP` и выбираем из списка `0.0.0.0/0`
- Кликаем на `Save rules`
- Открываем в браузере http://3.68.71.93/ и видим сайт
- Останавливаем приложение - `Ctrl + C`

### Создание docker образа
- Создал простенькое приложение на php (Просто текст)
- Создал Dockerfile
- Собрал образ докера
- Создал аккаунт в Docker Hub
- Создал репозиторий в Docker Hub
- Запушал готовый образ в Docker Hub
- Удалил образ у себя локально
- Запустил контейнер с образом из Docker Hub
  - docker run -it -p 1234:80 66233599134422234992/k8sphp
- Container -> Pod -> Deployment -> Service ->
  1. Pod - объект в котором работают один или больше Containers
  2. Deployment - Сэт одинаковых Pods, нужен для Auto Scaling и для обновления Container Image, держит минимальное количество работающих Pods.
  3. Service - предоставляет доступ к Deployment через
     - ClusterIP
     - NodePort
     - LoadBalancer
     - ExternalName
  4. Nodes - Сервера где всё это работает
  5. Cluster - Логическое объединение Nodes
    

### Запуск k8s
- Создал 3 сервера на https://labs.play-with-k8s.com/
- Создал мастер ноду и 2 воркера
- Подключил 2 ноды к мастеру 
  - `kubeadm join 192.168.0.31:6443 --token ypxzwt.9ksxmsohndkubl43 --discovery-token-ca-cert-hash sha256:6c16458c12995134b4516e5e4c90af88e4c0a46449dd54314f8e3e611d229c3e`
- На worker ноде выполнил команду 
  - kubectl run hello --generator=run-pod/v1 --image=66233599134422234992/k8sphp:latest --port=80
    
- kubectl describe pods hello
- curl ifconfig.co
- docker run -it -p 1234:80 66233599134422234992/k8sphp

Пятница:
- Прочитал https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/
- Нашел курсы по K8s https://kubernetes.io/training/

Главные темы для рассказа:
- AWS: EC2, EKS
- eksctl, kubectl
- K8s:
    - Компоненты
    - Спобоб работы
    - kube-apiserver (предоставляет K8S API, создает новые инстансы, распределяет нагрузку)
    - etcd (резервное хранилище Kubernetes для всех данных кластера)
    - kube-scheduler (отслеживает новые Pods, назначает Pod на ноду)
    - kube-controller-manager (следит за задачами, процессами, следит за нодами)
    - cloud-controller-manager (добавляет логику работы провайдера)
    - kubelet (есть в каждой ноде - следит за работой и исправностью контейнеров K8S)
    - kube-proxy (есть на каждой ноде - позволяет Pod обращаться с кластером)
- Утилиты для K8s
    - kubectl
    - kubeadm
    - minikube
    

### minikube (https://kubernetes.io/docs/tutorials/hello-minikube/#open-dashboard-with-url)
- minikube start
- kubectl create deployment hello-node --image=k8s.gcr.io/echoserver:1.4
- kubectl get deployments
- kubectl get pods
- kubectl get events
- kubectl config view
- kubectl expose deployment hello-node --type=LoadBalancer --port=8080
- 

### kubectl:
- kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
- kubectl create deployment hello --image=66233599134422234992/k8sphp:latest
- echo -e "\n\n\n\e[92mStarting Proxy. After starting it will not output a response. Please click the first Terminal Tab\n"; kubectl proxy
- export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}') echo Name of the Pod: $POD_NAME
  
### https://training.play-with-kubernetes.com/kubernetes-workshop/
- Инициализируем кластер `kubeadm init --apiserver-advertise-address $(hostname -i)`
- Подключаем ноду к кластеру `kubeadm join 192.168.0.8:6443 --token bv2yzn.ayaozewp3azbmufz --discovery-token-ca-cert-hash sha256:0875f5cac20e653e3a5ad97bba26f975ee475878af7131424548699ab022f55f`
- Создаем сеть `kubectl apply -n kube-system -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 |tr -d '\n')"`
- Скачиваем репозиторий для тестового приложения `git clone https://github.com/dockersamples/dockercoins`
- Смотрим список нод с подробным описание `kubectl get nodes -o wide`
- `kubectl get no -o yaml`
- Список доступных ресурсов на нодах `kubectl get nodes -o json | jq ".items[] | {name:.metadata.name} + .status.capacity"`
- Список серписов внутри кластера `kubectl get services` == `kubectl get svc`
- Список подов `kubectl get pods`
- Список пространств имен `kubectl get namespaces`
- Можем переключиться в другой namespace `kubectl -n kube-system get pods`
- Запускаем приложение `kubectl run pingpong --image alpine ping 8.8.8.8`
- `kubectl get all`
- Смотрим логи запущенного приложения `kubectl logs pod/pingpong`
- Смотрим последние логи в real time приложения `kubectl logs deploy/pingpong --tail 1 --follow`
- `kubectl scale pod/pingpong --replicas 8`
- Смотрим поды в real time `kubectl get pods -w`
- Смотрим логи `kubectl logs -l run=pingpong --tail 1`
- Удаляем под `kubectl delete pod/pingpong`
- Запускаем кучу приложений `kubectl run elastic --image=elasticsearch:2 --replicas=4`

  [comment]: <> (- `kubectl create deployment pinpong --image=alpine ping 8.8.8.8`)


#### The Kubernetes logic (its “brains”) is a collection of services:

- the API server (our point of entry to everything!)
- core services like the scheduler and controller manager
- etcd (a highly available key/value store; the “database” of Kubernetes)

Together, these services form what is called the “master”

#### Kubernetes architecture: the nodes

The nodes executing our containers run another collection of services:

- a container Engine (typically Docker)
- kubelet (the “node agent”)
- kube-proxy (a necessary but not sufficient network component)

Nodes were formerly called “minions”