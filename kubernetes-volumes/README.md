# Svyatoslav2792_platform
Svyatoslav2792 Platform repository
ДЗ 4
Применение StatefulSet
    "kubectl apply -f .\minio-statefulset.yaml"
Применение Headless Service
    "kubectl apply -f .\minio-headless-service.yaml"
Проверка работы MinIO
    "kubectl get statefulsets"
        NAME    READY   AGE
        minio   1/1     5m27s
    "kubectl get pods"
        NAME                             READY   STATUS    RESTARTS        AGE
        minio-0                          1/1     Running   0               6m1s
    "kubectl get pvc"
        NAME           STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
        data-minio-0   Bound    pvc-1f1c3c9e-72c7-4c20-ad1b-b5ff4eb2a6d7   10Gi       RWO            standard       6m26s
    "kubectl get pv"
        NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   REASON   AGE
        pvc-1f1c3c9e-72c7-4c20-ad1b-b5ff4eb2a6d7   10Gi       RWO            Delete           Bound    default/data-minio-0   standard                6m49s
    "kubectl describe pod/minio-0"
        Name:             minio-0
        Namespace:        default
        Priority:         0
        Service Account:  default
        Node:             minikube/192.168.49.2
        Start Time:       Fri, 02 Jun 2023 09:08:48 +0300
        Labels:           app=minio
                          controller-revision-hash=minio-dfc57cb8
                          statefulset.kubernetes.io/pod-name=minio-0
        Annotations:      <none>
        Status:           Running
        IP:               10.244.0.251
        IPs:
          IP:           10.244.0.251
        Controlled By:  StatefulSet/minio
        Containers:
          minio:
            Container ID:  docker://324422519a972ff14d1b69186fae8727f01c756e74db965fc43a2bfc2dddfc31
            Image:         minio/minio:RELEASE.2019-07-10T00-34-56Z
            Image ID:      docker-pullable://minio/minio@sha256:ccdbb297318f763dc1110d5168c8d45863c98ff1f0d7095a90be3b31a150ac6f
            Port:          9000/TCP
            Host Port:     0/TCP
            Args:
              server
              /data
            State:          Running
              Started:      Fri, 02 Jun 2023 09:09:16 +0300
            Ready:          True
            Restart Count:  0
            Liveness:       http-get http://:9000/minio/health/live delay=120s timeout=1s period=20s #success=1 #failure=3
            Environment:
              MINIO_ACCESS_KEY:  minio
              MINIO_SECRET_KEY:  minio123
            Mounts:
              /data from data (rw)
              /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2bm59 (ro)
        Conditions:
          Type              Status
          Initialized       True
          Ready             True
          ContainersReady   True
          PodScheduled      True
        Volumes:
          data:
            Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
            ClaimName:  data-minio-0
            ReadOnly:   false
          kube-api-access-2bm59:
            Type:                    Projected (a volume that contains injected data from multiple sources)
            TokenExpirationSeconds:  3607
            ConfigMapName:           kube-root-ca.crt
            ConfigMapOptional:       <nil>
            DownwardAPI:             true
        QoS Class:                   BestEffort
        Node-Selectors:              <none>
        Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                                     node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
        Events:
          Type     Reason            Age    From               Message
          ----     ------            ----   ----               -------
          Warning  FailedScheduling  7m58s  default-scheduler  0/1 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/1 nodes are available: 1 No preemption victims found for incoming pod..
          Normal   Scheduled         7m56s  default-scheduler  Successfully assigned default/minio-0 to minikube
          Normal   Pulling           7m55s  kubelet            Pulling image "minio/minio:RELEASE.2019-07-10T00-34-56Z"
          Normal   Pulled            7m28s  kubelet            Successfully pulled image "minio/minio:RELEASE.2019-07-10T00-34-56Z" in 27.248108172s (27.248130194s including waiting)
          Normal   Created           7m28s  kubelet            Created container minio
          Normal   Started           7m28s  kubelet            Started container minio
   
Задание со * 
   Создаем файл minio-secrets.yaml с секретами в base64
   "kubectl apply -f .\minio-secrets.yaml"
   Добавляем в minio-statefulset.yaml секреты и волт
   "kubectl apply -f .\minio-statefulset.yaml"
   "kubectl describe pod/minio-0"
        ...
        Environment:
          MINIO_ACCESS_KEY:  <set to the key 'MINIO_ACCESS_KEY' in secret 'minio-secrets'>  Optional: false
          MINIO_SECRET_KEY:  <set to the key 'MINIO_SECRET_KEY' in secret 'minio-secrets'>  Optional: false
        Mounts:
          /data from data (rw)
          /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-9lswl (ro)
        ...
    "kubectl exec minio-0  -it -- /bin/sh"
        "printenv" и видим в переменных окружения наши секреты внутри пода
            ...
            MINIO_ACCESS_KEY=minio
            MINIO_SECRET_KEY_FILE=secret_key
            MINIO_SECRET_KEY=minio123
            ...