# Svyatoslav2792_platform
Svyatoslav2792 Platform repository
ДЗ 8

Helm

    Создаем кластер с одним узлом в yc, а пока он стартует- устанавливаем Yandex Cloud CLI.
    curl -sSL https://storage.yandexcloud.net/yandexcloud-yc/install.sh | bash
        Downloading yc 0.108.1
          % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                         Dload  Upload   Total   Spent    Left  Speed
        100  100M  100  100M    0     0  25.3M      0  0:00:03  0:00:03 --:--:-- 25.4M
        Yandex Cloud CLI 0.108.1 windows/amd64
        
    "yc init" и настраиваем, в итоге получаем 
    "yc config list"
        token: y0_Ag********************************Io
        cloud-id: b1gugsb01**********
        folder-id: b1gio9a93**********
        compute-default-zone: ru-central1-a
    
    Добавляем учетные данные кластера Managed Service for Kubernetes в конфигурационный файл kubectl:
    "yc managed-kubernetes cluster get-credentials k8stemplates --external"
    
    Проверяем конфиг "kubectl config view"
    apiVersion: v1
    clusters:
    ...
    - cluster:
        certificate-authority-data: DATA+OMITTED
        server: https://51.250.72.100
      name: yc-managed-k8s-cata5mgl0smj7e1t8plf
    ...
    
    Затем создаем в полученном кластере группу узлов с одним узлом.
    
    Nginx-ingress устанавливаем через Helm 
        "kubectl create namespace nginx-ingress"
        "helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx"
        "helm repo update"
        "helm install ingress-nginx ingress-nginx/ingress-nginx --namespace=nginx-ingress"
        "kubectl get all --namespace nginx-ingress"
            NAME                                          READY   STATUS    RESTARTS   AGE
            pod/ingress-nginx-controller-78d54fbd-8jr6g   0/1     Pending   0          8s
            
            NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)                      AGE
            service/ingress-nginx-controller             LoadBalancer   10.96.250.35   51.250.85.21   80:30349/TCP,443:31037/TCP   8s
            service/ingress-nginx-controller-admission   ClusterIP      10.96.212.74   <none>         443/TCP                      8s
            
            NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
            deployment.apps/ingress-nginx-controller   0/1     1            0           8s
            
            NAME                                                DESIRED   CURRENT   READY   AGE
            replicaset.apps/ingress-nginx-controller-78d54fbd   1         1         0       8s
            
    Certmanager устанавливаем через Helm а и настраиваем
    
    "kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.12.2/cert-manager.crds.yaml"
        customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io created
        customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io created
        customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io created
        customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io created
        customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io created
        customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io created
    
    "kubectl apply -f .\cert-manager\clusterissuer.yml"
        clusterissuer.cert-manager.io/letsencrypt created
        
    Проверяем "kubectl describe clusterissuers -n cert-manager"
        Name:         letsencrypt
        Namespace:
        Labels:       <none>
        Annotations:  <none>
        API Version:  cert-manager.io/v1
        Kind:         ClusterIssuer
        Metadata:
          Creation Timestamp:  2023-07-25T17:54:13Z
          Generation:          1
          Resource Version:    9620
          UID:                 c26565e1-ea7b-4b3a-8c05-e9d7361205b7
        Spec:
          Acme:
            Email:  svyatoslav2792@yandex.ru
            Private Key Secret Ref:
              Name:  letsencrypt
            Server:  https://acme-v02.api.letsencrypt.org/directory
            Solvers:
              http01:
                Ingress:
                  Class:  nginx
        Events:           <none>
    
    "kubectl create ns chartmuseum" 
    "helm repo add chartmuseum https://chartmuseum.github.io/charts"
    "helm upgrade --install chartmuseum chartmuseum/chartmuseum --wait --namespace=chartmuseum --version=3.10.0 -f chartmuseum/values.yaml"
        Ловим ошибку 
            Error: UPGRADE FAILED: failed to create resource: Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": failed to call webhook: Post "https://ingress-nginx-controller-admission.nginx-ingress.svc:443/networking/v1/ingresses?timeout=10s": dial tcp 10.96.212.74:443: connect: connection refused
        Чтобы пофиксить- нужно удалить вебхук (мб кому поможет)
            "kubectl get validatingwebhookconfigurations"
                NAME                      WEBHOOKS   AGE
                ingress-nginx-admission   1          31m
            "kubectl delete validatingwebhookconfigurations ingress-nginx-admission"
                validatingwebhookconfiguration.admissionregistration.k8s.io "ingress-nginx-admission" deleted
    Проверяем что все запустилось "helm ls -n chartmuseum"
        NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
        chartmuseum     chartmuseum     4               2023-07-25 21:12:30.367339 +0300 MSK    deployed        chartmuseum-3.10.0      0.16.0   
    Проверяем доступность по адресу https://chartmuseum.51.250.85.21.nip.io
    