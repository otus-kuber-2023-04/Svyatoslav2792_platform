# Svyatoslav2792_platform
Svyatoslav2792 Platform repository
ДЗ 3

Добавление проверок Pod
    Вносим изменения в файл kubernetesintro/web-pod.yml 
          # --- BEGIN --
          readinessProbe:          # Добавим проверку готовности
            httpGet:               # веб-сервера отдавать
              path: /index.html    # контент
              port: 80
          # --- END --
    "kubectl apply -f .\web-pod.yaml"
    "kubectl get pod/web"
        NAME   READY   STATUS    RESTARTS   AGE
        web    0/1     Running   0          97s
    "kubectl describe pod/web"
        ...
        Conditions:
          Type              Status
          Initialized       True
          Ready             False
          ContainersReady   False
          PodScheduled      True
        ...
        Warning  Unhealthy  8s (x12 over 94s)  kubelet            Readiness probe failed: Get "http://10.244.0.209:80/index.html": dial tcp 10.244.0.209:80: connect: connection refused
    Добавляем в файл kubernetesintro/web-pod.yml 
        livenessProbe: 
            tcpSocket: { port: 8000 }
    "kubectl apply -f .\web-pod.yaml"
    
Создание Deployment
    "kubectl delete pod/web --grace-period=0 --force"
    Создаем новый манифест web-deploy.yaml и всносим правки, чтобы все работало
    "kubectl apply -f web-deploy.yaml"
    "kubectl describe deployment web"
        ...
        Conditions:
          Type           Status  Reason
          ----           ------  ------
          Available      True    MinimumReplicasAvailable
        ...
    Добавляем 
          type: RollingUpdate
          rollingUpdate:
            maxUnavailable: 0
            maxSurge: 100%
    Применяем манифест "kubectl apply -f web-deploy.yaml" и наблюдаем за процессом "kubectl get events -watch"

Создание Service
    Создадим манифест для нашего сервиса web-svc-cip.yaml
    "kubectl apply -f web-svc-cip.yaml"
    "kubectl get services"
        NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
        ...
        web-svc-cip     ClusterIP   10.108.251.98    <none>        80/TCP     36s
    Подключаемся к миникьюбу "minikube ssh" 
        "curl http://10.108.251.98/index.html" - работает
        "ping 10.108.251.98" - пинга нет
        "ip addr show" - наш айпи отсутствует
        "sudo -i iptables --list -nv -t nat" находим наш кластерный адрес
            Chain KUBE-SERVICES (2 references)
             pkts bytes target     prot opt in     out     source               destination
                ...
                1    60 KUBE-SVC-6CZTMAROCN3AQODZ  tcp  --  *      *       0.0.0.0/0            10.108.251.98        /* default/web-svc-cip cluster IP */ tcp dpt:80
            Chain KUBE-SVC-6CZTMAROCN3AQODZ (1 references)
             pkts bytes target     prot opt in     out     source               destination
                1    60 KUBE-MARK-MASQ  tcp  --  *      *      !10.244.0.0/16        10.108.251.98        /* default/web-svc-cip cluster IP */ tcp dpt:80

Включение IPVS
    "kubectl --namespace kube-system edit configmap/kube-proxy"
    Вносим изменения в соотв. с заданием
         ipvs:     
            strictARP: true   
         mode: "ipvs"
    "kubectl --namespace kube-system delete pod --selector='k8s-app=kube-proxy'"
    "minikube ssh" 
    "kubectl --namespace kube-system exec kube-proxy-<POD> kube-proxy -cleanup"
    Поставил себе нано, так как вимом пользоваться "sudo apt update && sudo apt install -y nano" невозможно и поменял параметры
    "sudo -i iptables-restore /tmp/iptables.cleanup"
    
Установка MetalLB
    Тут все оп инструкции, за исключением того, что указанный в ДЗ манифест падает с ошибкой 
        resource mapping not found for name: "controller" namespace: "metallb-system" from "https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml": no matches for kind "PodSecurityPolicy" in version "policy/v1beta1"
        ensure CRDs are installed first
        resource mapping not found for name: "speaker" namespace: "metallb-system" from "https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml": no matches for kind "PodSecurityPolicy" in version "policy/v1beta1"
        ensure CRDs are installed first
    Поэтому пришлось применить другой "kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.7/config/manifests/metallb-native.yaml"
    "kubectl --namespace metallb-system get all"
        NAME                              READY   STATUS    RESTARTS   AGE
        pod/controller-577b5bdfcc-ppjpr   1/1     Running   0          2m27s
        pod/speaker-cx6ft                 1/1     Running   0          2m26s
        NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
        service/webhook-service   ClusterIP   10.96.189.158   <none>        443/TCP   2m27s
        NAME                     DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
        daemonset.apps/speaker   1         1         1       1            1           kubernetes.io/os=linux   17m
        NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
        deployment.apps/controller   1/1     1            1           17m
        NAME                                    DESIRED   CURRENT   READY   AGE
        replicaset.apps/controller-5759df545c   0         0         0       17m
        replicaset.apps/controller-577b5bdfcc   1         1         1       2m27s
        replicaset.apps/controller-bdf98b979    0         0         0       9m29s
    Создаем манифест metallb-config.yaml и "kubectl apply -f metallb-config.yaml"
    Делаем копию файла web-svc-cip.yaml в web-svc-lb.yaml и ставим LoadBalancer "kubectl apply -f .\web-svc-lb.yaml"
    IP:                       10.106.146.215
    Затем пробуем зайти на веб интерфейс и не можем, находим адрес своего миникуба "minikube ssh" -> "ip addr show eth0"
        inet 192.168.49.2/24 brd 192.168.49.255 scope global eth0 
    Поигрался, в итоге не взлетело, бросил..
Создание Ingress
    "kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/deploy.yaml"
    Создаем манифест nodeport.yaml и "kubectl apply -f .\nodeport.yaml"
Подключение приложение Web к Ingress
    Ккопируйкм web-svc-cip.yaml в web-svc-headless.yaml, меняем параметры и "kubectl apply -f .\web-svc-headless.yaml"
Создание правил Ingress
    Создаем манифест web-ingress.yaml и "kubectl apply -f .\web-ingress.yaml", 
    проверяем корректность заполнения "kubectl describe ingress/web"