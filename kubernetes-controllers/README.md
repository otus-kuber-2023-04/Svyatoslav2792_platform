# Svyatoslav2792_platform
Svyatoslav2792 Platform repository
ДЗ 2
Kind
 установлен и создан кластер "kind create cluster --config kind-config.yaml"
 
ReplicaSet
    при применении "kubectl apply -f frontend-replicaset.yaml" получаем ошибку:
        The ReplicaSet "frontend" is invalid:
        * spec.selector: Required value
        * spec.template.metadata.labels: Invalid value: map[string]string{"app":"frontend"}: `selector` does not match template `labels`
    обавляем в спек
          selector:
            matchLabels:
              app: frontend
    и получаем "kubectl get pods -l app=frontend":
        NAME             READY   STATUS    RESTARTS   AGE
        frontend-vbx2k   1/1     Running   0          9s
    скейлим поды "kubectl scale replicaset frontend --replicas=3" и получаем "kubectl get rs frontend":
        NAME       DESIRED   CURRENT   READY   AGE
        frontend   3         3         3       4m5s
    дропаем поды "kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w" и видим как восстанавливаются 
        NAME             READY   STATUS              RESTARTS   AGE
        frontend-kjkdd   0/1     ContainerCreating   0          3s
        frontend-lhf7g   0/1     ContainerCreating   0          4s
        frontend-sxhzt   0/1     ContainerCreating   0          4s
        frontend-kjkdd   1/1     Running             0          5s
        frontend-lhf7g   1/1     Running             0          8s
        frontend-sxhzt   1/1     Running             0          10s
    снова применяем манифест "kubectl apply -f frontend-replicaset.yaml" и видим что под снова один:
        NAME             READY   STATUS    RESTARTS   AGE
        frontend-kjkdd   1/1     Running   0          95s
    меняем в манифесте replicas: 1 -> 3, применяем и видим:
        NAME             READY   STATUS    RESTARTS   AGE
        frontend-6s552   1/1     Running   0          7s
        frontend-gpqvp   1/1     Running   0          7s
        frontend-kjkdd   1/1     Running   0          2m29s
        
Обновление ReplicaSet
    "docker pull fastfighter92/frontend:latest"
    "docker tag fastfighter92/frontend:latest fastfighter92/frontend:v0.0.2"
    "docker push fastfighter92/frontend:v0.0.2"
    меняем версию образа в frontend-replicaset.yaml на fastfighter92/frontend:v0.0.2
        "kubectl apply -f .\frontend-replicaset.yaml"
        "kubectl apply -f frontend-replicaset.yaml | kubectl get pods -l app=frontend -w"
            NAME             READY   STATUS    RESTARTS   AGE
            frontend-b8jmn   1/1     Running   0          3m18s
            frontend-n9llw   1/1     Running   0          3m18s
            frontend-tc8mq   1/1     Running   0          3m18s     
    кажется и правда ничего не произошло
         "kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'" выдает нам
            fastfighter92/frontend:v0.0.2
          kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'
          fastfighter92/frontend:latest fastfighter92/frontend:latest fastfighter92/frontend:latest
          Происходит это потомучто репликасет не отвлеживает изменения в манифесте для уже развернутых подов
         Удаляем все поды "kubectl delete pod --all" и видим "kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'"
            fastfighter92/frontend:v0.0.2 fastfighter92/frontend:v0.0.2 fastfighter92/frontend:v0.0.2 образ изменился, на указанный в манифесте
         
Deployment
            "docker build -t fastfighter92/paymentservice:v0.0.1 ."
            "docker build -t fastfighter92/paymentservice:v0.0.2 ."
            "docker push fastfighter92/paymentservice:v0.0.1"
            "docker push fastfighter92/paymentservice:v0.0.2"
        внесены правки в paymentservice-deployment.yaml
            "kubectl apply -f .\paymentservice-deployment.yaml"
            "kubectl get pods"
                NAME                              READY   STATUS    RESTARTS   AGE
                payment-service-56755b4f8-77485   1/1     Running   0          4s
                payment-service-56755b4f8-mdphj   1/1     Running   0          4s
                payment-service-56755b4f8-mwx59   1/1     Running   0          4s
            "kubectl get rs"
                NAME                        DESIRED   CURRENT   READY   AGE
                payment-service-56755b4f8   3         3         1       51s
        Обновляем версию до v0.0.2
            kubectl apply -f paymentservice-deployment.yaml | kubectl get pods -l app=payment-service -w
            NAME                              READY   STATUS    RESTARTS   AGE
            paymentservice-74599ffb7b-bnmkp   0/1     Pending   0          0s
            paymentservice-74599ffb7b-hwnkj   0/1     Pending   0          0s
            paymentservice-74599ffb7b-j8n2m   0/1     Pending   0          0s
            paymentservice-74599ffb7b-hwnkj   0/1     ContainerCreating   0          1s
            paymentservice-74599ffb7b-bnmkp   0/1     ContainerCreating   0          1s
            paymentservice-74599ffb7b-j8n2m   0/1     ContainerCreating   0          1s
            paymentservice-74599ffb7b-bnmkp   1/1     Running             0          5s
            paymentservice-74599ffb7b-j8n2m   1/1     Running             0          5s
            paymentservice-74599ffb7b-hwnkj   1/1     Running             0          5s
        "kubectl get rs"
            NAME                        DESIRED   CURRENT   READY   AGE
            payment-service-84655997d   3         3         3       10s
            payment-service-96b4dd597   0         0         0       13h
        "kubectl rollout history deployment payment-service"
            deployment.apps/payment-service
            REVISION  CHANGE-CAUSE
            1         <none>
            2         <none>
            
Deployment | Задание со *
        Аналог blue-green:
        Добавляем 
          strategy:
            type: RollingUpdate
            rollingUpdate:
              maxSurge: 100%
              maxUnavailable: 0
        "kubectl rollout restart deployment/payment-service-bg"
            deployment.apps/payment-service-bg restarted
        "kubectl get pods"                                     
            NAME                                  READY   STATUS              RESTARTS   AGE
            payment-service-bg-6cfd95f7d5-5tt8g   0/1     ContainerCreating   0          3s
            payment-service-bg-6cfd95f7d5-gvknm   0/1     ContainerCreating   0          3s
            payment-service-bg-6cfd95f7d5-n6n2g   0/1     ContainerCreating   0          3s
            payment-service-bg-76664459fd-czg29   1/1     Running             0          4m35s
            payment-service-bg-76664459fd-czx4c   1/1     Running             0          4m35s
            payment-service-bg-76664459fd-nq5gr   1/1     Running             0          4m35s
    Reverse Rolling Update:
      strategy:
        type: RollingUpdate
        rollingUpdate:
          maxSurge: 1
          maxUnavailable: 1
      "kubectl apply -f paymentservice-deployment-reverse.yaml"
          NAME                                       READY   STATUS    RESTARTS   AGE
          payment-service-reverse-5b85b75dd9-87pzg   1/1     Running   0          5s
          payment-service-reverse-5b85b75dd9-r9jjb   1/1     Running   0          5s
          payment-service-reverse-5b85b75dd9-xrdzl   1/1     Running   0          5s
      "kubectl rollout restart deployment/payment-service-reverse"
          NAME                                       READY   STATUS              RESTARTS   AGE
          payment-service-reverse-5b85b75dd9-87pzg   1/1     Running             0          10s
          payment-service-reverse-5b85b75dd9-r9jjb   1/1     Terminating         0          10s
          payment-service-reverse-5b85b75dd9-xrdzl   1/1     Running             0          10s
          payment-service-reverse-d799b4c44-f29p2    1/1     Running             0          1s
          payment-service-reverse-d799b4c44-ljrv5    0/1     ContainerCreating   0          1s 
    Probes 
        добавляем
            readinessProbe:
              initialDelaySeconds: 10
              httpGet:
                path: "/_healthz"
                port: 8080
                httpHeaders:
                  - name: "Cookie"
                    value: "shop_session-id=x-readiness-probe"   
        после изменения path: "/_healthz" на path: "/_health"
          Warning  Unhealthy  8s (x2 over 18s)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 404   

DaemonSet | Задание со *
    был найден образец манифеста и развернут демонсет  "kubectl apply -f .\node-exporter-daemonset.yaml"
    далее прокинут порт "kubectl port-forward prometheus-node-exporter-vt8zx 9100:9100" и появилась возможность просмотреть метрики "curl localhost:9100/metrics"