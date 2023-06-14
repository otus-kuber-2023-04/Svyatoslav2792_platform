# Svyatoslav2792_platform
Svyatoslav2792 Platform repository
ДЗ 5

Security
    Скачиваем и станавливаем krew " .\krew install krew"
    Далее устанавливаем rbac-view "kubectl krew install rbac-view" и запускаем "kubectl rbac-view"
    
   task01
        Создаем манифесты bob.yaml и dave.yaml, применяем  "kubectl apply -f .\01-bob.yaml"
              serviceaccount/bob created
              clusterrolebinding.rbac.authorization.k8s.io/bob-cluster-admin created
        "kubectl apply -f .\02-dave.yaml"
              serviceaccount/dave created
        "kubectl get serviceaccount"
            NAME      SECRETS   AGE
            bob       0         17m
            dave      0         16m
            default   0         17d
        "kubectl get rolebindings,clusterrolebindings --all-namespaces -o wide"
            NAMESPACE   NAME                                                           ROLE                      AGE USERS GROUPS SERVICEACCOUNTS
                        clusterrolebinding.rbac.authorization.k8s.io/bob-cluster-admin ClusterRole/cluster-admin 17m              default/bob
        
   task02
        Создаем манифесты 01-prometheus.yaml, 02-carol.yaml и 03-pod-reader.yaml, применяем по порядку
         "kubectl apply -f .\01-prometheus.yaml"
            namespace/prometheus created
         "kubectl apply -f .\02-carol.yaml"
            serviceaccount/carol created
         "kubectl apply -f .\03-pod-reader.yaml"
            clusterrole.rbac.authorization.k8s.io/pod-reader created
            clusterrolebinding.rbac.authorization.k8s.io/rb-prometheus created
         "kubectl -n prometheus get serviceaccount"
             NAME      SECRETS   AGE
             carol     0         12m
             default   0         13m
         "kubectl get rolebindings,clusterrolebindings --all-namespaces -o wide"
             NAMESPACE   NAME                                                       ROLE                   AGE   USERS GROUPS                            SERVICEACCOUNTS
                         clusterrolebinding.rbac.authorization.k8s.io/rb-prometheus ClusterRole/pod-reader 3m26s       system:serviceaccounts:prometheus
   task03
         Создаем манифесты 01-dev.yaml, 02-jane.yaml и 03-ken.yaml, применяем по порядку  
            "kubectl apply -f .\01-dev.yaml"  
                namespace/dev created
            "kubectl apply -f .\02-jane.yaml"  
                serviceaccount/jane created
                rolebinding.rbac.authorization.k8s.io/rb-jane created
            "kubectl apply -f .\03-ken.yaml"  
                serviceaccount/ken created
                rolebinding.rbac.authorization.k8s.io/rb-ken created
            "kubectl -n dev get serviceaccount"
                NAME      SECRETS   AGE
                default   0         52s
                jane      0         41s
                ken       0         27s
            "kubectl get rolebindings,clusterrolebindings --all-namespaces -o custom-columns='KIND:kind,NAMESPACE:metadata.namespace,NAME:metadata.name,SERVICE_ACCOUNTS:subjects'"
                RoleBinding          dev              rb-jane                                                [map[kind:ServiceAccount name:jane namespace:dev]]
                RoleBinding          dev              rb-ken                                                 [map[kind:ServiceAccount name:ken namespace:dev]]
            