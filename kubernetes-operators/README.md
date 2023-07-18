# Svyatoslav2792_platform
Svyatoslav2792 Platform repository
ДЗ 7
    Создаем crd.yaml и cr.yaml
        "kubectl apply -f .\deploy\crd.yml" и ловим ошибку
            error: resource mapping not found for name: "mysqls.otus.homework" namespace: "" from ".\\deploy\\crd.yml":
             no matches for kind "CustomResourceDefinition" in version "apiextensions.k8s.io/v1.25"
            ensure CRDs are installed first
            порылся в сети и нашел что надо понизить версию кубера (у меня стоял v1.26.3), поэтому создаем новый профиль в 
            миникубе "minikube start -p minikube2 --kubernetes-version=v1.16.0"
            и после второй попытки применения манифекста получаем 
                customresourcedefinition.apiextensions.k8s.io/mysqls.otus.homework created
            Ура! Надеюсь кому-нибудь поможет, времени заняло много)
        "kubectl apply -f .\deploy\cr.yml"
            mysql.otus.homework/mysql-instance created
        "kubectl get crd"
            NAME                   CREATED AT
            mysqls.otus.homework   2023-07-18T17:57:00Z
        "kubectl get mysqls.otus.homework"
            NAME             AGE
            mysql-instance   57s
    Разработка оператора через kopf (python)
        Устанавливаем зависимости
            "pip install kopf"
            "pip install kubernetes"
            "pip install pyyaml" 
            "pip install jinja2"
        Копируем готовые файлы.
        Запускаем "kopf run .\build\mysql-operator.py"
        Проверяем, что все работает
        Создаем докерфайл, собираем образ и пушим     
            docker build -t fastfighter92/kubernetes-operators:v0.0.1 .
            docker push fastfighter92/kubernetes-operators:v0.0.1
         docker build -t fastfighter92/kubernetes-operators:v0.0.1 .
    Деплой оператора
        Создаем в папке kubernetes-operator/deploy: 
            -deploy-operator.yml    (тут подставляем свой образ fastfighter92/kubernetes-operators:v0.0.1)
            -role.yml
            -role-binding.yml
            -service-account.yml
        Применяем манифесты
            "kubectl apply -f service-account.yml"
            "kubectl apply -f .\deploy\service-account.yml"
            "kubectl apply -f .\deploy\role.yml"
            "kubectl apply -f .\deploy\role-binding.yml"
            "kubectl apply -f .\deploy\deploy-operator.yml"
        Проверяем, что все работает
            "kubectl get jobs"
                NAME                         COMPLETIONS   DURATION   AGE 
                backup-mysql-instance-job    1/1           2s         44m 
                restore-mysql-instance-job   1/1           3m12s       45m 
        Добвляем данные
            export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath="{.items[*].metadata.name}") 
            kubectl exec -it $MYSQLPOD -- mysql -u root  -potuspassword -e "CREATE TABLE test ( id smallint unsigned not null auto_increment, name varchar(20) not null, constraint pk_example primary key (id) );" otus-database
             kubectl exec -it $MYSQLPOD -- mysql -potuspassword  -e "INSERT INTO test ( id, name ) VALUES ( null, 'some data' );" otus-database kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "INSERT INTO test ( id, name ) VALUES ( null, 'some data-2' );" otus-database 
        Получаем данные
            export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath="{.items[*].metadata.name}") 
            kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database 
                +----+-------------+
                | id | name        | 
                +----+-------------+
                |  1 | some data   |
                |  2 | some data-2 |
                +----+-------------+
    Задания со звездочкой* не осилил, чтото развалилось, не смог понять что именно, ошибки питона, забил...