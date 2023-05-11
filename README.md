ДЗ 1
Задание
Разберитесь почему все pod в namespace kube-system восстановились после удаления. Укажите причину в описании PR
Получем список подов
PS C:\WINDOWS\system32> kubectl get pods -n kube-system
NAME                               READY   STATUS    RESTARTS   AGE
coredns-787d4945fb-swz46           1/1     Running   0          30m
etcd-minikube                      1/1     Running   1          30m
kube-apiserver-minikube            1/1     Running   1          30m
kube-controller-manager-minikube   1/1     Running   1          30m
kube-proxy-bwt5s                   1/1     Running   0          30m
kube-scheduler-minikube            1/1     Running   1          30m

Получаем описание
kubectl get pods -n kube-system -o yaml
Из описания видим, что 
coredns-787d4945fb-swz46
kind: ReplicaSet
Цель ReplicaSet — поддерживать стабильный набор подов реплик, работающих в любой момент времени. Таким образом, он часто используется, чтобы гарантировать доступность определенного количества идентичных подов.

kube-apiserver-minikube,kube-controller-manager-minikube, kube-scheduler-minikube, etcd-minikube
kind: Node
Компоненты узла запускаются на каждом узле, поддерживая работающие модули и обеспечивая среду выполнения Kubernetes

kube-proxy-bwt5s 
kind: DaemonSet
DaemonSet гарантирует , что все (или некоторые) узлы запускают копию пода. По мере добавления узлов в кластер к ним добавляются поды. Когда узлы удаляются из кластера, эти поды удаляются сборщиком мусора. Удаление DaemonSet приведет к очистке созданных им подов.

Далее:
Был создан образ с nginx, для которого создан соответствующий докерфайл, сбилжено и выложено на докерхаб
fastfighter92/kubernetes-intro. (ихначально сделал на flask со стартовой страницей, но тесты посыпались с ошибкой File "/tmp/pytest/lib/python3.10/site-packages/kubetest/objects/clusterrolebinding.py", line 30, in ClusterRoleBinding
    'rbac.authorization.k8s.io/v1alpha1': client.RbacAuthorizationV1alpha1Api,
AttributeError: module 'kubernetes.client' has no attribute 'RbacAuthorizationV1alpha1Api'. Did you mean: 'RbacAuthorizationV1Api'?
Error: Process completed with exit code 1.)
Далее
kubectl apply -f web-pod.yaml
kubectl port-forward --address 0.0.0.0 pod/web 8000:8000 
после чего на http://localhost:8000 можно увидеть 403 от nginx.

Задание*
kubectl run frontend --image fastfighter92/frontend --restart=Never 
PS D:\otus_kuber\2\microservices-demo\src\frontend> kubectl logs frontend
	{"message":"Tracing disabled.","severity":"info","timestamp":"2023-05-11T10:43:52.094536888Z"}
	{"message":"Profiling disabled.","severity":"info","timestamp":"2023-05-11T10:43:52.09470067Z"}
	panic: environment variable "PRODUCT_CATALOG_SERVICE_ADDR" not set
	goroutine 1 [running]:
	main.mustMapEnv(0xc00068c0c0, {0xc0771a, 0x1c})
			/src/main.go:208 +0xb9
	main.main()
			/src/main.go:124 +0x5be
Очевидно ошибка возникает из-за отсутcтвия переменных окружения. 

Поэтому добавляем в frontend-pod-healthy.yaml недостающие переменные и запускаем, теперь все в порядке.
kubectl apply -f .\frontend-pod-healthy.yaml
 PS D:\otus_kuber\2\Svyatoslav2792_platform> kubectl get pods
	NAME       READY   STATUS    RESTARTS      AGE
	frontend   1/1     Running   0             56s
	web        1/1     Running   1 (19m ago)   71m
	