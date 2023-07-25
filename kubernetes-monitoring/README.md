# Svyatoslav2792_platform
Svyatoslav2792 Platform repository
ДЗ 6

Мониторинг сервиса в кластере k8s
    Создаем неймспейс
        "kubectl create namespace monitoring"
    Ставим prometheus-stack (helm ставим отдельно, расписывать не буду, там просто)
        "helm repo add prometheus-community https://prometheus-community.github.io/helm-charts"
        "helm upgrade --install kube-prometheus-stack prometheus-community/kube-prometheus-stack -n monitoring"
    Делаем новый конфиг и докерфайл для нджинкса с роутом для метрик, билдим и заливаем имэдж в репу
        "docker build -t fastfighter92/kubernetes-monitoring:v0.0.1 ."
        "docker push fastfighter92/kubernetes-monitoring:v0.0.1"
    Разворачиваем
        "kubectl apply -f nginx-deployment.yaml"
    Ставим prometheus-nginx-exporter
        "helm upgrade --install prometheus-nginx-exporter prometheus-community/prometheus-nginx-exporter -n monitoring -f nginx-exporter.yaml"
    Смотрим, что все нужное взлетело 
        "kubectl get --namespace monitoring pods"
            NAME                                                        READY   STATUS    RESTARTS   AGE
            alertmanager-kube-prometheus-stack-alertmanager-0           2/2     Running   0          69m
            kube-prometheus-stack-grafana-77cc8c9b78-rgqx2              3/3     Running   0          69m
            kube-prometheus-stack-kube-state-metrics-57fbc885c8-8wrqk   1/1     Running   0          69m
            kube-prometheus-stack-operator-6d559875d6-jls2m             1/1     Running   0          69m
            kube-prometheus-stack-prometheus-node-exporter-qskjf        0/1     Pending   0          69m
            prometheus-kube-prometheus-stack-prometheus-0               2/2     Running   0          69m
            prometheus-nginx-exporter-7dc6ccddb8-8hdg4                  1/1     Running   0          67m
    Пробрасываем порты
        -Тут сами метрики собираются
            "kubectl --namespace monitoring port-forward prometheus-nginx-exporter-7dc6ccddb8-8hdg4 8080:9113"
        -Тут графана
            "kubectl --namespace monitoring port-forward kube-prometheus-stack-grafana-77cc8c9b78-rgqx2 8080:3000"
            (П.С. чтобы авторизоваться в графане нужны креды, валяются тут "kubectl get secret --namespace monitoring kube-prometheus-stack-grafana -o yaml", естественно необходимо декодировать из base64)
             	data:
             	  admin-password: cHJvbS1vcGVyYXRvcg==
             	  admin-user: YWRtaW4=
    Заходим в графану по адресу http://127.0.0.1:8080/
        В настройках красоты пока-что не силен, сделал график по метрике "nginx_connections_handled"- красивое)
        ![](/assets/k8s-monitoring-6.png)
        <image src="assets/k8s-monitoring-6.png" alt="Текст с описанием картинки">