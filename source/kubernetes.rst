Kubernetes documentation
========================


Features
^^^^^^^^

Для того, чтобы использовать вместо ``kubectl`` просто ``k`` необходимо добавить в файл ~/.bashrc

.. code:: console

        alias k=kubectl

YAML file structure
^^^^^^^^^^^^^^^^^^^

Full structure
~~~~~~~~~~~~~~

Metadata - метаданные

Spec - описание содержимого поды

Status - текущая информация о работающем модуле.

Details structure
~~~~~~~~~~~~~~~~~

Присоединение к нодам с определенными метками (например ``gpu = true`` )

.. code:: console

        spec:
          nodeSelector:
            gpu: "true"

Command
^^^^^^^

Перенаправление локального порта 8888 на порт 8080 модуля test

.. code:: console

        $ kubectll port-forward test 8888:8080

Команда для выполнения curl запроса изнутри какой-то поды. В этой команде (`--`) используется для сигнала kubectl об окончании командных параметров

.. code:: console

        $ kubectl exec podname -- curl -s http://ip

Команда на получение всех endpoints для службы

.. code:: console

        $ kubectl get endpoints kubia

Вывод всех классов хранилищ

.. code:: console

        $ kubectl get sc

Для того, чтобы записать свою команду в историю ревизий

.. code:: console

        $ kubectl create -f file.yaml --record



Help and information
^^^^^^^^^^^^^^^^^^^^

Команды для помощи для создания yaml файла

.. code:: console

        $ kubectl explain pod.spec

Вывод информации о модуле в виде yaml файла

.. code:: console

        $ kubectl get po name -o yaml

Labels (Метки)
^^^^^^^^^^^^^^

Информация о подe с учетом определенных меток

.. code:: console

        $ kubectl get po -L creation_method,env

Изменение меток существующих модулей (test)

.. code:: console

        $ kubectl label po test creation_method=manual
        $ kubectl label po test env=debig --overwrite

Селекторы меток 

.. code:: console

        $ kubectl get pods -l creation_method=manual

Вывод всех модулей, которые включают метку env

.. code:: console

        $ kubectl get pods -l env

Вывод модулей, которые не включают метку env

.. code:: console

        $ kubectl get po -l '!env'

Создать метку gpu для виртуальной машины slave

.. code:: console

        $ kubectl label node slave gpu=true

Получение списка узлов по меткам

.. code:: console

        $ kubectl get nodes -l gpu=true

Удаление модулей с помощью селектора меток

.. code:: console

        $ kubectl delete pods -l creation-method=manual

Annotation
^^^^^^^^^^

Добавление аннотации в существующий объект

.. code:: console

        $ kubectl annotate pod test mycompany.com/someannotation="foo bar"


Namespace
^^^^^^^^^

Для создания namespace требуется создать yaml файл и применить его

.. code:: console

        apiVersion: v1
        kind: Namespace
        metadata:
          name: custom-namespace


Либо создать пространство имен можно следующим образом

.. code:: console

        $ kubectl create namespace custom-namespace

Удаление пространства имен

.. code:: console

        $ kubectl delete ns custom-namespace

Liveness
^^^^^^^^

Добавление проверки живучести в модуль

.. code:: console

        apiVersion: v1
        kind: pod
        metadata:
          name: test
        spec:
          containers:
          - image: test
            name: test
            livenessProbe:
              httpGet: 
                path: /
                port: 8080
              initialDelaySeconds: 15 # (Перед первой проверкой ждем 15 секунд)

Получение лога приложения аварийного контейнера

.. code:: console

        $ kubectl logs test --previous
             
Replication controller
^^^^^^^^^^^^^^^^^^^^^^

Контроллер репликаций состоит из трех основных частей
- селектор меток, определяющий, какие модули находятся в области действия контроллера репликации
- количества меток, указывающее на требуемое количество модулей, которые должны быть запущены
- шаблон модуля, используемый при создании новых реплик модуля

Пример создания контроллера репликаций 

.. code:: console

        apiVersion: v1
        kind: ReplicationController
        metadata:
          name: test
        spec:
          replicas: 3
          selector:
            app: test
          template:
            metadata:
              labels:
                app: test
            spec:
              containers:
              - name: test
                image: test
                ports:
                - containerPort: 8080


Удаление контроллера репликаций без затрагивания под (они перестанут быть управляемыми)

.. code:: console

        $ kubectl delete rc test --cascade=false

ReplicaSet
^^^^^^^^^^

Репликасет отличается от контроллера репликаций более гибким использованием метрик
Привет YAML файла в ReplicaSet

.. code:: console

        apiVersion: apps/v1beta2
        kind: ReplicaSet
        metadata:
          name: kubia
        spec:
          replicas: 3
          selector:
            matchLabels:
              app: kubia
          template:
            metadata:
              labels:
                app: kubia
            spec:
              containers:
              - name: kubia
                image: luksa/kubia

Команда для отображения всех  ReplicaSet и информации о ReplicaSet

.. code:: console

        $ kubectl get rs
        $ kubectl describe rs name

Пример селектора с помощью matchExpressions

.. code:: console

        selector: 
          matchExpressions:
            - key: app
              operator: In
              values:
              - kubia

В селектор можно добавлять следующие выражения:

- `In` - значение метки должно совпадать с одним из указанных `values`;

- `NotIn` - значение метки не должно совпадать ни с одним из указанных `values`;

- `Exists` - модуль(пода) должна содержать метку с указанным ключом;

- `DoesNotExist` - модуль(пода) не должна содержать метку с указанным ключом.

DaemonSet
^^^^^^^^^

DaemonSet требуется для того, чтобы точно определять на каких нодах и в каком количестве должен быть развернута та или иная пода. Типичный пример его использования - это сборщик логов. Таким образом DaemonSet является аналогом ReplicaSet с пропуском планировщика.

Пример YAML файла для DaemonSet, который, например, должен использоваться на виртуалках (нодах), у которых есть метка ``disk: ssd``.

.. code:: console

        apiVersion: app/v1beta2
        kind: DaemonSet
        metadata:
          name: ssd-monitor
        spec:
          selector:
            matchLabels:
              app: ssd-monitor
          template:
            metadata:
              labels:
                app: ssd-monitor
            spec:
              nodeSelector:
                disk: ssd
              container: 
              - name: main
                image: luksa/ssd-monitor

Job
^^^

Задачи нужны для того, чтобы запускать единичные процессы, после их успешного завершения задача будет удалена. Job может быть сконфигугрирована таким образом, чтобы параллельно или последовательно выполялись определенные задачи. 

Пример YAML файла для Job

.. code:: console
        apiVersion: batch/v1
        kind: Job
        metadata:
          name: batch-job
        spec:
          template:
            metadata:
              labels: 
                app: batch-job
            spec: 
              restartPolicy: OnFailure
              containers:
              - name: main
                image: image

Пример YAML файла для того, чтобы Job выполнялось несколько раз и было разрешено запускать Job в несколько потоков параллельно

.. code:: console
        apiVersion: batch/v1
        kind: Job
        metadata:
          name: batch-job
        spec:
          completions: 5
          parallelism: 2
          selector:
            matchLabels:
              app: batch-job
          template:
            metadata:
              labels:
                app: batch-job
            spec:
              containers:
              - name: main
                image: testimage 
              
                
CronJon
^^^^^^^

Job запускает свои модули немедленно при создании ресурса Job. Чтобы запускать задачи по расписанию - используется CronJob. Пример YAML файла для создания CronJob

.. code:: console

        apiVersion: apps/v1beta1
        kind: CronJob
        metadata:
          name: batch-job-every-fifteen-minutes
        spec:
          schedule: "0,15,30,45 * * * *"
          startingDeadlineSeconds: 15
          jobTemplate:
            spec:
              template:
                metadata:
                  labels:
                    app: periodic-batch-job
                spec:
                  restartPolicy: OnFailure
                  containers:
                  - name: main
                    image: luksa/batch-job


Services
^^^^^^^^

Сервсиы необходимы для того, чтобы сформировать единую постоянную точку входа в группу модулей, предоставляющих одну и то жуе службу.

Пример YAML файла для создания service

.. code:: console

        apiVersion: v1
        kind: Service
        metadata:
          name: kubia
        spec:
          ports: 
          - port: 80
            targetPort: 8080
          selector:
            app: kubia

Данная служба принимает подключение по 80 порту и маршрутизирует каждое подключение на порт 8080 оного из модулей, у которого есть отметка app=kubia

Для того, чтобы все запросы, сделанные определенным клиентом, каждый раз перенаправлялись в один и тот же модуль - то свойству sessionAffinity службы можно присвлить значение ClientIp

.. code:: console

        apiVersion: v1
        kind: Service
        spec:
          sessionAffinity: ClientIp

Это заставляет служебный прокси перенаправлять все запросы, исходящие от того же клиентского IP адреса на ту же поду.  Kubernetes поддерживает тоглько два типа сохранения сессии - None и ClientIp.

Службы могут поддерживать доступ к нескольким портам. Пример YAML файла, который поддерживает это приведен ниже

.. code:: console

        apiVersion: v1
        kind: Service
        metadata:
          name: kubia
        spec:
          ports:
          - name: http
            port: 80
            targetPort: 8080
          - name: https
            port:433
            targetPort:8443
          selector:
            app: kubia

Кроме того можно ссылаться не только на номера портов, но также на их имена. Предположим, что в поде определены уже порты на примере

.. code:: console
        
        ...
        kind: Pod
        spec:
          containers:
          - name: kubia
            ports:
            - name: http
              containerPort: 8080
            - name: https
              containerPort: 8443
        ...

        apiVersion: v1
        kind: Service
        spec:
          ports:
          - name: http
            port: 8080
            targetPort: http
          - name: https
            port: 8433
            targetPort: https


Службы без обозначенной точки входа
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Чтобы создать service без обозначенной точки входа (Headless), требуется присвоить полю clusterIp значение None. 

.. code:: console

        apiVersion: v1
        kind: Service
        metadata: 
          name: kubia-headless
        spec:
          clusterIp: None
          ports:
          - port: 80
            targetPort: 8080
          selector:
            app: kubia

Headless service нужны для того, чтобы клиенты подключались непосредственно к модулям, а не через служебный прокси.




DNS
^^^

Пода с названием `kube-dns` запускает DNS сервер, для использования которого автоматически настравиваются все оставльные модули. Любой DNS запрос будет обрабатываться собственным DNS-сервером Kubernetes, который знает все службы в нашей системе.

Если требуется подключиться к бэкэнд базе данных - надо открыть подключение со следующим доменным именем

``backend-database.default.svc.cluster.local``
где ``backend-database`` - название service, ``default`` - обозначает namespace, ``svc.cluster.local`` - настраиваемый доменный суффикс кластера, используемый во всем именах локальных служб. 

          
Service endpoints setting
^^^^^^^^^^^^^^^^^^^^^^^^^

Иногда бывает необходимым настраивать список endpoints для service вручную.
Пример YAML файла

.. code:: console

        apiVersion: v1
        kind: Service
        metadata:
          name: external-service
        spec:
          ports:
          - port: 8080

Endpoints представляют из себя отдельный ресурс, а не атрибут службы. И поэтому, если Endpoints не был создан автоматически, его надо создать вручную

.. code:: console

        apiVersion: v1
        kind: Endpoints
        metadata: 
          name: external-service
        subsets:
          - adresses:
            - ip: 11.11.11.11
            - ip: 22.22.22.22
            ports:
            - port: 80

Таким образом имя Endpoint сопадает с названием соответствующего сервиса. После того, как service и endpoints будут отправлены на сервер, service будет готов к использования, как любой service с селектором модулей. Контейнеры, созданные после создания service будут включать переменные окружения для service, и все подключения с парой IP:port будут балансироваться между конечными точками службы.

Так же вместо предоставления доступа внешней служюе путем ручной настройки конечных точек службы более простой способ позовляет ссылаться на внешнюю службу по ее полностью квалифицированному доменному имени. Например, если общедоступный API имеется по адресу `test.com`, то мы можем определить service, который указывает на него

.. code:: console

        apiVersion: v1
        kind: Service
        metadata:
          name: external-service
        spec:
          type: ExternalName
          externalName: test.com
          ports:
          - port: 80

Надо отметить, что в качестве externalName не может быть использован IP.

Access to service outside the cluset
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Для того, чтобы внешний клиент мог использовать службу внутри кластера существуют следующие способы

- Присвоить типу service значение NodePort. Каждая нода кластера открывает порт и перенаправляет трафик в базовую службу. Service доступен не только через внутренний IP и порт кластера, но и через выделенный порт на всех узлах

- Присвоить типу service значение LoadBalancer, расширение типа NodePort - это делает службу доступной через выделенный балансировщик нагрузок, зарезервированный из облачной инфраструктуры. Балансировщик нагрузок перенаправляет трафик на порт node во всех nodes. Внешний клиент подключается через IP адрес балансировщика нагрузки

- Создать ресурс Ingress, который работает на уровне HTTP

NodePort
~~~~~~~~

К service NodePort можно получить доступ не только через внутренний кластреный IP адрес, но и через IP адресс любого узла.

Пример создания service NodePort

.. code:: console

        apiVersion: v1
        kind: Service
        metadata:
          name: kubia-nodeport
        spec:
          type: NodePort
          ports: 
          - port: 80
            targetPort: 8080
            nodePort: 30123
          selectror:
            app: kubia

LoadBalancer
~~~~~~~~~~~~

Балансировщик нагрузку имеет свой IP и все запросы будут идти через него. Пример YAML файла

.. code:: console

        apiVersion: v1
        kind: Service
        metadata:
          name: kubia-loadbalancer
        spec:
          type: LoadBalancer
          ports:
          - port:80
            targetPort: 8080
          selectror:
            app: kubia

Ingress
~~~~~~~

Пример YAML файла для Ingress

.. code:: console

        apiVersion: extensions/v1beta1
        kind: Ingress
        metadata
          name: kubia
        spec:
          rules:
          - host: kubia.example.com
            http:
              paths:
              - path: /
                backend:
                  serviceName: kubia-nodeport
                  servicePort: 80

Проверка готовности
^^^^^^^^^^^^^^^^^^^

Проверка готовности отличается от Liveness тем, что если пода не прошла проверку готовности, то в таком случае она не удаляется, а удаляется Endpoint.
Пример YAML файла с проверкой готовности

.. code:: console

        apiVersion: v1
        kind: ReplicationController
        ...
        spec:
          ...
          template:
            spec:
              containers: 
              - name: kubia
                image: luksa/kubia
                readinessProbe:
                  exec:
                    command:
                    - ls
                    - var/ready
          ...

В этом примере проверка готовности будет переодически выполнять команду ls /var/ready внутри контейнера. 


Тома
^^^^

Использование тома emptyDir
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Этот том начинается как пустой каталог. Приложение запущенное внутри модуля, может записывать любые файлы, которые ему нужны. Когда пода умирает - содержимое тома удаляется. Полезен для обмена файлами между контейнерами, запущенными на одной поде. 

Пример ииспользования emptyDir в поде. В качестве веб-сервера будет выступаить Nginx, для создания HTML-контента будет использоваться команда `fortune` системы Unix. Будет создан скрипт, который будет вызывать команду `fortune` каждый 10 секунд и сохранять результат в файле index.html. 

.. code:: console

        apiVersion: v1
        kind: Pod
        metadata: 
          name: fortune
        spec:
          containers: 
          - image: luksa/fortune
            name: html-generator
            volumeMounts:
            - name: html
              mountPath: /var/htdocs
          - image: nginx:alpine
            name: web-server
            volumeMounts: 
            - name: html
              mountPath: /usr/share/nginx/html
              readOnly: true
            ports:
            - containerPort: 80
              protocol: TCP
          volumes:
          - name: html
            empryDir: {}

Чтобы создать  `emptyDir` в файловой системе `tmpfs` (в памяти, а не на диске), достаточно присвить свойство `medium: Memory` 

.. code:: console

        volumes:
        - name: html
          emptyDir:
            medium: Memory

Использование тома gitRepo
~~~~~~~~~~~~~~~~~~~~~~~~~~

По сути то же самое, что и emptyDir, но только в том клонируется проект на гите. Важно отметить, что можно клонировать только открытые репозитории, с приватными подобный механизм не работает. 

.. code:: console

        apiVersion: v1
        kind: Pod
        metadata:
          name: gitrepo-volume-pod
        spec:
          containers:
          - image: nginx:alpine
            name: web-server
            volumeMounts:
            - name: html
              mountPath: /usr/share/nginx/html
              readOnly: true
            ports:
            - port: 80
              protocol: TCP
          volumes: 
          - name: html
            gitRepo:
              repository: https://github.com/...
              revision: master
              directory: .
            
Использование тома hotPath
~~~~~~~~~~~~~~~~~~~~~~~~~~

Том hostPath указывает на определенный файл или каталог в файловой системе узла. Модули, работающие на одном узле и использующие один и тот же путь в томе hostPath видят одни и те же файлы. Надо отметить, что при удалении поды, файлы в hostPath остаются неизменными. Чаще всего сюда просто складываются логи. 

Использование постоянного диска GCE Persistent Disk
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Для этого необходимо вначале создать диск в той же зоне, где размещен и кластер.Здесь приведен пример с Google, аналогично, если речь идет про Azure надо использовать azureDisk.  Пример YAML файла

.. code:: console

        apiVersion: v1
        kind: Pod
        metadata: 
          name: mongodb
        spec:
          volumes: 
          - name: mongodb-disk
            gcePersistentDisk:
              pdName: mongodb
              fsType: ext4
          containers:
          - image: mongo
            name: mongodb
            volumeMounts:
            - name: mongodb-data
              mountPath: /data/db
            ports:
            - containerPort: 27017
              protocol: TCP

PersistentVolume and PersistentVolumeChaim
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Согласно глобальной идеалогии Kubernetes, не очень хорошо, когда надо указывать тип диска и тд. Для таких целей существует PersistentVolume и PersistentChaim. Весь алгоритм добавления тома устроен следующим образом.

- Админ создает сетевое хранилище

- Админ создает том PV и потом отправляет дескриптор PV в API Kubernetes. 

- Пользователь создает заявку PVC. 

- Kubernetes находит PV адекватного размера и связывает заявку с томом PV. 

- Пользователь создает поду с томом, ссылающуюся на заявку PVC.

Создание ресурса PersistentVolume делается по примеру следующего YAML файла

.. code:: console

        apiVersion: v1
        kind: PersistentVolume
        metadata:
          name: mongodb-pv
        spec:
          capacity:
            storage: 1Gi
          acessModes:
          - ReadWriteOnce
          - ReadOnlyMany
          persistentVolumeReclaimPolicy: Retain #после высвобождения заявки PersistentVolume должен быть сохранен
          gcePersistentDisk: #(каким диском поддерживается)
            pdName: mongodb
            fsType: ext4 

Теперь требуется создать заявку PersistentVolumeChaim. Пример такой заявки

.. code:: console

       apiVersion: v1
       kind: PersistentVolumeChaim
       metadata:
         name: mongodb-pvc
       spec:
         resources:
           requests:
             storage: 1Gi
         accessModes:
         - ReadWriteOnce
         storageClassName: ""

после того, как выполнить команду ``kubectl get pvc`` в поле AccessModes может быть несколько режимов доступа

- RWO - только одна нода может монтировать том для чтения
 
- ROX - несколько нод могут монтировать том для чтения

- RWX - несколько нод могут монтировать том как для чтения, так и для записи.

Кроме того можно использовать заяку PersistentVolumeChaim внутри YAML файла, описывающего поду. Пример

.. code:: console

        apiVersion: v1
        kind: Pod
        metadata:
          name: mongodb
        spec:
          containers:
          - image: mongo
            name: mongodb
            volumeMounts:
            - name: mongodb-data
              mountPath: /data/db
            ports: 
            - containerPort: 27017
              protocol: TCP
          volumes:
          - name: mongodb-data
            persistentVolumeClaim:
              claimName: mongodb-pvc

BestPractice является следующая схема. Администратор вместо того, чтобы создавать PersistentVolume может развернуть поставщика PersistentVolume и порделить несколько объектов StorageClass, чтобы позволить пользователям выбрать, какой тип ресурса PersistentVolume им больше всего подходит. Вместо предварительно резервирования кластера, админ может развренуть развернуть поставщика ресурсов PersistentVolume и определить несколько ресурсов StorageClass и позволить системе созадвать новый PersistentVolume всякий раз, когда один из них запрашивается с помощью заявки PersistentVolumeClaim. 

.. code:: console

        apiVersion: storage.k8s.io/v1
        kind: StorageClass
        metadata:
          name: fast
        provisioner: kubernetes.io/gce-pd # плагин тома, используемый для резервирования ресурса PV
        parameters:
          type: pd-ssd
          zone: europe-west1-b # параметры, передаваемыем поставщику

После создания ресурса StorageClass пользователи в своих заявках PersistentVolumeClaim могут ссылаться на класс хранилища по имени. Пример

.. code:: console

        apiVersion: v1
        kind: PersistentVolumeClaim
        metadata:
          name: mongodb-pvc
        spec:
          storageClassName: fast
          resources:
            requests:
              storage: 100Mi
          accessModes:
          - ReadWriteOnce




Deployment
^^^^^^^^^^

Пример deloyments файла 

.. code:: console

        apiVersion: apps/v1beta1
        kind: Deployment
        metadata:
          name: kubia
        spec:
          replicas: 3
          template:
            metadata:
              name: kubia
              labels:
                app: kubia
            spec:
              containers:
              - image: luksa/kubia:v1
              name: nodejs

Для того, чтобы поменять образ надо запустить команду

.. code:: console

        $ kubectl set image deployment kubia nodejs=luksa/kubia:v2

Откат к передыдущей версии осуществляется с помощью команды

.. code:: console

        $ kubectl rollout undo deployment kubia


Вывод истории выкатываний версий

.. code:: console

        $ kubectl rollout history deployment kubia

Откат к определенной версии 

.. code:: console

        $ kubectl rollout undo deployment --to-revision=1

Чтобы запустить канареечное развертывание (только часть модулей обновляем) необходимо выполнить ряд команд

.. code:: console

        $ kubectl set image deployment kubia nodejs=luksa/kubia:v4
        $ kubectl rollout pause deployment kubia
        $ kubectl rollout resume deployment kubia

        
maxSurge и maxUnavailable
~~~~~~~~~~~~~~~~~~~~~~~~~

На количество модулей, заменяемых одновременно во время deployment влияют 2 свойства. Пример

.. code:: console

        spec:
          strategy:
            rollingUpdate:
              maxSurge: 1
              maxUnavailable: 0
            type: RollingUpdate

maxSurge определяет, скольким экземплярам модуля вы позволяете существавать выше требуемого количества реплик. 

maxUnavailable определяет, сколько экземпляров модуля может быть недоступно относительно требуемого количества реплик. 

minReadySeconds позвляет задерживать развертывание на определенное количество секунд

Развертывание с промощью проверки готовности
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Пример YAML файла

.. code:: console

        apiVersion: apps/v1beta1
        kind: Deployment
        metadata:
          name: kubia
        spec:
          replicas: 3
          minReadySeconds: 10
          strategy:
            rollingUpdate
              maxSurge: 1
              maxUnavailable: 0
            type: RollingUpdate
          template:
            metadata:
              name: kubia
              labels:
                app: kubia
            spec:
              containers:
              - image: some/image
              name: nodejs
              readinessProbe: #(проверка готовности которая будет выполняться каждую секунду)
                periodSeconds: 1
                httpGet:
                  path: /
                  port: 8080

StatefulSet
^^^^^^^^^^^

Если у нас имеется набор ReplicaSet, то они все будут использовать одну заявку PersistentVolumeClaim. Требутеся сделать нечто, что управляла бы своим хранилищем. То есть сейчас рассматривается задача запуска множества реплик с отдельным хранилищем для каждой реплики. 

Каждому модулю присваивается порядковый индекс, который в дальнейшем используется, чтобы произвести имя и хост модуля и закрепить за модулем хранилище

Для того, чтобы сделать масштабирование вниз, требуется удалять заявку вручую. 

          
SRV
~~~

Записи SRV используюся для указания на хосты и порты серверов, представляющих конкретный service.  Для того, чтобы получить список записей SRV требуется запустить

.. code:: console

        $ kubectl run -it srvlookup --image=tutum/dnsutils --rm --restart=Never -- dig SRV kubia.default.svc.cluster.local

Данная команда запускается одноразовый модуль с именем srvlookup. Этот модуль запускает единственный контейнер tutum/dnsutils и выполняет команду 

Общаться модули между собой могут посредством вот такой команды. Чтобы обновить StatefulSet требуется удалять старые поды. Для того, чтобы рандомно кого-то вызывать - надо создать сервис 

.. code:: console

        apiVersion: v1
        kind: Service
        metadata: 
          name: kubia-public
        spec:
          selector:
            app: kubia
          ports:
          - port: 80
            targetPort: 8080




Внутреннее устройство Kubernetes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Архитектура
^^^^^^^^^^^

Master - то, что управляет всем кластером и заставляет его функционировать. Состоит из

- etcd (распределенное постоянное хранилище)

- server API

- scheduler (планировщик)

- controller Manager (менеджер контроллеров)

Nodes - узлы кластера, состоит из

- Kubelet agent

- kube-proxy

- Docker, rkt and etc

Компоненты системы Kubernetes напрямую взаимодействуют только с сервером API. Только сервер API взаимодействует с хранилищем etcd. Когда используется kubectl API подключается к агенту kubelet. 

etcd - хранит все изменения, конфигурационные файлы, версии и тд

serverAPI является API, с помощью которой можно вносить изменения в конфигурационные файлы, проверять состояние тех или иных модулей. В основном он просто хранит ресуры в хранилище и уведомляет клиентов об изменении. 

sheduler - планировщик, который при создании нового модуля должен определить узел, на котором требуется ему развернуться. 

controller manager - менеджер контроллеров, который включает

- replication controller

- replicaSeet, daemonSet, job

- Deployment

- StatefulSet

- контроллер узла

- контроллер службы Service

- контроллер конечных точек Endpoints

- контроллер пространства Namespaces

- контроллер постоянного тома Persistent Volume

То есть для каждого сервиса присутствует контроллер, который может его создать. Это активные компоненты Kubernetes, которые и выполняют работу по развертыванию ресурсов. 

Менеджеров контроллеров в случае ReplicaSet, ReplicationController, DaemonSet, Job проверяет через API состояние модулей и если что-то надо изменить,  то они отправляют определение модулей на сервер API, поручая агенту Kubelet создать и запустить контейнер. 

Контроллер Deployment обеспечивает синхронизацию фактического состояния с требуемым состояним, указанном в соответствующем объекте API Deployment. Контроллер при этом выполняте развертывание новой версии каждый раз при изменении объекта развертыванияю Это делается путем создания реплик. Опять же, никакие модули напрямую не создаются. 

Контроллер StatefulSet наоборот, может создавать  экземпляры и управлять заявками для каждого экземпляра модуля. 

Контроллер узла управляет ресурсами узла. Контроллер службы. 


Интермодульное взаимодействие
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Система Kubernetes не требует, чтобы вы использовали определенную сетевую технлогию, однако все модули должны взаимодействовать друг с другом независимо от того, на каком они узле расположены. Между модулями не должно быть преобразование сетевых адресов. 

Перед запуском инфраструктурного контейнера для контейнера создается пара виртуальных Ethernet интерфейсов. Один интерфейс пары остается в пространстве имен хоста, другой превращается в пространство сетевых имен контейнера и переименовывается в eth0. Интерфейс в пространстве сетевых имен присоединен к сетевому мосту, который сконфигурирован для использования средой выполнения контейнера. Интерфейсу eth0 назначается IP адрес из диапазона адресов моста. 

Таким образом, все контейнеры на узле подключены к одному мосту и поэтому могут взаимодействовать друг с другом. Для разных узлов эти мосты надо каким-то образом подсоединить. Контейнер на одном из узлов отправляет пакет контейнеры на другом узле, пакет проходит вначале veth пару, затем через мост к физическому адаптеру, потом у другому адаптеру узла, через мост другого узла и опять через пару veth. Это работает, когда узлы подсоединены к одному сетевому коммутатору без маршрутизаторов между ними. Поэтой причине делают SDN, которая делает узлы такими, как будто они подключены к одному и тому же коммутатору, независимо от фактической базовой топологии сети, какой бы сложной она ни была. 

CNI(Container network interface)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

CNI позволяет конифгурировать систему Kubernetes для использования любого существующего плагина. Эти плагины включают Calico, Flannel, Romana, Weave Net и тд. Этот плаген разворачивается на всех узлах и говорит, чтобы агент Kubelet был запущен с параметром --network-plugin=cni.

Как реализованы службы
~~~~~~~~~~~~~~~~~~~~~~

Все, что связано со службами занимается процесс kube-proxy, работающий на каждом узле. Каждая служба имеет свой собственный стабильный IP адрес и порт. Клиенты используют службу, подключаясь к этому адресу и порту. 

При создании службы на сервере API ей немедленно назначается виртуальный адрес. API уведомляет всех агентов kube-proxy о создании новой службы. Каждая kube-proxy делает эту службу адресуемой на узле, где она запущена. Это делается путем настройки правил iptables, которые гарантируют, что каждый пакет, предназначенный для пары IP/порт перехвачен и целевой адрес модифицирован. 

Объект Endpoints содержит пары IP/порт всех модулей, который поддерживают службу. Поэтому kube-proxy следит за всеми объектами Endpoints. Все происходит примерно следующим образом. Модуль запрашивает определенную службу, имея целевой адрес этой службы, служба редиректит на один из модулей, kube-proxy заменяет 

Защита сервера API Kubernetes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Существует несколько плагинов аутентификации. Они получают идентичность клиента следующими способоми. 
- Из сертификата клиента

- Из токена аутентификации, передаваемого в заголовке http

- В результате обыкновенной HTTP-аутентификации.

Пользователи могут быть двух типов, реальные люди и модули. Модули используют механизм учетных записей служб (ресурсы)

Как обычные пользователи, учетные записи ServiceAccount могут принадлежать нескольким группам. Модуль может использовать учетную запись только из того же пространства имен. 

Одним из плагинов авторизации является плагин управления ролевым доступом (RBAC) Role based access control. Это нужно для того, чтобы если используются модули , которым не нужно читать метаданные кластера, должны работать  в соответствии с огрниченной учетной записью, которая не позволяет им извлекать и менять ресуры в кластере. Если учетная запись ServiceAccount аннотируется kubernetes.io/enforce-mountable-secrets="true", то любые использующие ее модули могуть монтировать только монтируемые серкреты service-account. 

Пример yaml файла для service account для выгрузки докер образов

.. code:: console

        apiVersion: v1
        kind: ServiceAccount
        metadata:
          name: my-service-account
        imagePullSecrets:
        - name: my-dockerhub-secrets

Секреты учетной записи для выгрузки образов в отличие от монтируемых секретов не определяют какие секреты автоматически добавляются во все модули с помощью учетной записи служюы. Это позволяет не добавлять секреты в каждый модуль отдельно. 

Пример создания модуля, использующего индивидуальную запись службы.

.. code:: console

        apiVersion: v1
        kind: Pod
        metadata:
          name: curl-custom-sa
        spec:
          serviceAccountName: foo
          containers:
          - naame: main
            image: tutum/curl
            command: ["sleep", "9999"]
          - name: ambassador
            image: some/image

Для того, чтобы убедиться, что токен индивидуально натсроен учетной записи нужно выполнить команду

.. code:: console

        $ kubectl exec -it curl-custom-sa -c main cat /var/run/secrets/kubernetes.io/serviceaccount/token

Защита кластера с помощью управления ролевым доступом
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

RBAC требуется для того, чтобы понять, какие команды разрешено выполнять клиенту, а какие нет. 
То есть  ServiceAccount связан с определенными ролями, а RBAC их разрешает или не разрешает выполнять в опредленных ресурсах. 

Правило авторизации RBAC настраиваются с помощью четырех ресурсов, которые можно сгруппировать в 2 группы.

- роли Role и кластерной роли ClusterRole, которые задают какие глаголы могут выполняться на ресурсах.

- привязка ролей RoleBinding и привязка кластерных ролерй ClusterRoleBinding, которые привязывают роли к определенным пользователям, группам или ServiceAccount. 

Роли определяют что можно делать, привязки определяют кто может делать. Разница между Role vs ClusterRole и RoleBinding vs ClusterRoleBinding заключается в том, что все, что с Cluster не обязано быть в одном пространстве имен и вообще им пофиг на пространство имен. Но все, что без кластера тем не менн могут ссылать на учетные записи из других namespace. 

Если создать 2 модуля в разных namespace, зайти в один и выполнить запрос с получением всех сервисов в другом namespace, то ничего не выйдет, так как процесс получил запрос, отправил его на сервер API, аутентифицируясь как учетная запись ServiceAccount по умолчанию в пространстве имен foo и сервер API сказал, что учетной записи службы это не разрешено. Попробуем это исправить.

.. code:: console

        apiVersion: rbac.authorization.k8s.io/v1
        kind: Role
        metadata:
          namespace: foo
          name: service-reader
        rules:
        - apiGroups: [""]
          verbs: ["get", "list"]
          resources: ["services"]

Далее требуется привзять роль к субъектам. Это выполняется с помощью

.. code:: console

        $ kubectl create rolebinding test --role=service-reader --serviceaccount=foo:default -n foo

Это можно выполнить с помощью команды

.. code:: console

        apiVersion: rbac.authorization.k8s.io/v1
        kind: RoleBinding
        metadata:
          name: test
          namespace: foo
        roleRef:
          apiGroup: rbac.authorization.k8s.io
          kind: Role
          name: service-readre
        subjectsL
        - kind: ServiceAccount
          name: default
          namespace: foo
        - kind: ServiceAccount
          namespace: default
          namespace: bar

Пример ClusterRole

.. code:: console

        apiVersion: rbac.authorization.k8s.io/v1
        kind: ClusterRole
        metadata:
          name: pv-reader
          resourceVersion: "39992"
          ...
        rules:
        - apiGroups:
          - ""
          resources:
          - persistentVolumes
          verbs:
          - get
          - list

Привязка ClusterRole

.. code:: console

        apiVeriosn: rbac.authorization.k8s.io/v1
        kind: ClusterRoleBinding
        metadata:
          name: pv-test
          ...
        roleRef:
          apiGroup: rbac.authorization.k8s.io
          kind: ClusterRole
          name: pv-reader
        subjects:
        - kind: ServiceAccount
          name: default
          namespace: foo
          
Резюмирование комбинаций Role, ClusterRole, RoleBinding, ClusterRoleBinding
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ресурсы уровня кластера - ClusterRole, ClusterRoleBinding
Нересурсные URL пути  - ClusterRole, ClusterRoleBinding
Ресурсы простраства имен - ClusterRole, ClusterRoleBinding

Использование в модуле сетевого пространства имен узла
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Модуль может использовать сетевые интерфейсы узла, вместо того, чтобы иметь собственный набор. Для этого требуется сделать hostNetwork=true

.. code:: console

        apiVersion: v1
        kind: Pod
        metadata:
          name: pod-with-host-network
        spec:
          hostNetwork: true
          containers:
          - name: main
            image: alpine
            command: ["/bin/sleep", "99999"]

После применения команды

.. code:: console

        $ kubectl exec pod-with-host-network ifconfig

Можно увидеть сетевое пространство имен хоста(в частности все сетевые адаптеры узла)

Кроме того возможно привязывать модулям к порту в стандартном пространстве имен. Есть два свойства, hostPort и NodePort. Когда модуль использует hostPort подключение к порту узла перенаправляется к модулю на этом узле, когда в случае службы NodePort подключение к порту узла перенаправляется к случайному модулю (возможно, на другом узле). Примеры использования

.. code:: console

        apiVersion: v1
        kind: Pod
        metadata:
          name: kubia-hostport
        spec:
          containers:
          - image: some/image
            name: kubia
            ports:
            - continaerPort: 8080
              hostPort: 9000
              protocol: TCP

hostPort в основном нужно для предоставления системных служб, развернутых на  каждом узле. 

Использование PID и IPC узла
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Свойства hostPID и hostIPC секции spec аналогичны параметру hostNetwork. Если им задать значение true, то контейнеры модуля смогут использовать пространство PID и IPC узла. 

.. code:: console

        apiVersion: v1
        kind: Pod
        metadata: 
          name: pod-with-pid-and-ipc
        spec:
          hostPID: true
          hostIPC: true
          containers:
          - name: main
            image: alpine
            command: ["/bin/sleep", "999999"]

Конфигурация контекста безопасности контейнера
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Конфигугрировать безопасность можно через свойство securityCOntext. Конфигурирование контекста безопасности позволяет

- Указать пользователя, под которым будет выполняться процесс в контейнере

- Не допустить контейнеру выполняться в качестве root

- Запустить контейнер в привилигированном режиме, предоставив ему полный доступ к ядру узла

- Сконфигурировать тонко настроенные привилегии, добавляя или удаляя возможности. 

- Установить параметр SELinux

- Не дать контейнеру записывать в файловую систему контейнера

Выполнение контейнера от имени конкретного пользователя. 

.. code:: console

        apiVersion: v1
        kind: Pod
        metadata: 
          name: pod-as-user-guest
        spec:
          containers: 
          - name: main
            image: alpine
            command: ["/bin/sleep", "9999999"]
            securityContext:
              runAsUser: 405

405 - это код guest, поэтому контейнер будет запущен от имени guest

Если мы хотим указать, чтобы контейнер модуля выполнялся от имени пользователя без полномочий root - 

.. code:: console

        apiVersion: v1
        kind: Pod
        metadata:
          name: example
        spec: 
          containers:
          - name: main
            image: alpine
            command: ["/bin/sleep", "999999"]
            securityContext:
              runAsNonRoot: true

Выполнение модулей в привилегированном режиме
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Чтобы получить полный доступ к ядру узла контейнер модуля должен работать в привилегированном режиме. 
.. code:: console

        apiVersion: v1
        kind: Pod
        metadata:
          name: pod-privileged
        spec:
          containers:
          - name: main
            image: alpine
            command: ["/bin/slepp", "99999" ]
            securityContext:
              privileged: true


Чтобы не дать контейнеру поменять владельца файла в файловой системе надо добавить 

.. code:: console

        ...
        securityContext:
          capabiilities:
            drop:
            - CHOWN

Если нельзя контейнеру запись в файловую систему контейнера, это делается следующим образом

.. code:: console

        securityContext:
          readOnlyRootFilesystem: true
        volumeMounts:
        - name: my-volume
          mountPath: /volume
          readOnly: false

В такой конфигурации можно записывать только в /volume, так как там смонтирован том. 


Совместное использование томов, когда контейнеры запущены под разными пользователями
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: console

        apiVersion: v1
        kind: Pod
        metadata: 
          name: pod-test
        spec:
          securityContext:
            fsGroup: 555
            suplimentalGroups: [666, 777]
          containers:
          - name: first
            image: alpine
            command: ["/bin/sleep", "99999"]
            securityContext:
              runAsUser: 1111
            volumeMounts:
            - name: shared-volume
              mountPath: /volume
              readOnly: false
          - name: second
            image: alpine
            command: ["sleep", "9999"]
            securityContext:
              runAsUser: 2222
            volumeMounts:
            - name: shared-volume
              mountPath: /volume
              readOnly: false
          volumes:
          - name: shared-volume
            emptyDir:

Создание политики безопасности модуля, позволяющей развертывать привилегированные контейнеры 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: console

        apiVersion: extensions/v1beta1
        kind: PodSecurityPolicy
        metadata:
          name: privileged
        spec:
          privileged: true
          runAsUser:
            rule: RunAsAny
          fsGroup:
            rule: RunAsAny
          supplementalGroup:
            
Можно использовать RBAC для назначения разным пользователя разные PodSecurityPolicy. Для этого надо создать кластерные роли Clusterrole и указать различные политики. 

.. code:: console

        $ kubectl create clustertole psp-default --verb=use --resource=podsecuritypolicies --resource-name=default
        $ kubectl create clusterrole psp-privileged --verb=use --resource=podsecuritypolicies --resource-name=privileged
        $ kubectl create clusterrolebinding psp-aa-isers --clusterrole=psp-default --group=system:authenticated
        $ kubectl create clusterrolebinding psp-bob --clusterrole=psp-privileged --user=bob

Для создания пользователей используются команды

.. code:: console

        $ kubectl config set-credentials alice --username=alice --password=password
        $ kubectl config set-credentials bob --username=bob --password=password

Чтобы создавать пользователи от чего-то имени надо

.. code:: console

        $ kubectl --user alice create -f pod-privileged.yaml

NetworkPolicy
^^^^^^^^^^^^^

Чтобы разрешить подключаться к модулю только некоторых в пространстве имен

.. code:: console

        apiVersion: networking.k8s.io/v1
        kind: NetwotkPolicy
        metadata:
          name: postgres-netpolicy
        spec:
          podSelector:
            matchLabels:
              app: database
           ingress:
           - from:
             - podSelector:
                 matchLabels:
                   app: webserver
               ports: 
               - port: 5432

.. code:: console

        apiVersion: networking.k8s.io/v1
        kind: NetworkPolicy
        metadata:
          name: shoppingcart-netpolicy
        spec:
          podSelector:
            matchLabels:
              app: shopping-cart
          ingress:
          - from:
            - namespaceSelector:
                matchLabels:
                  tenant: manning
            ports:
            - port: 80


Управление вычислительными ресурсами модулей
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Создание модуля с ресурсными запросами
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Пример создания модуля с ограничениями по ЦП и памяти

.. code:: console

        apiVersion: v1
        kind: Pod
        metadata:
          name: request-pod
        spec:
          containers:
          - image: busybox
            command: ["dd", "if=/dev/zero", "of=/dev/null"]
            name: main
            resources:
              requests:
                cpu: 200m
                memory: 10Mi

Контейнер запрашивает 200 миллиядер и 10 мебибайт памяти оперативной памяти. Для того, чтобы посмотреть потребление ЦП процессом

.. code:: console

        $ kubectl exec -it request-pod top

Создание модуля с лимитами на ресурсы

.. code:: console

        apiVersion: v1
        kind: Pod
        metadata:
          name: limited-pod
        spec:
          containers:
          - image: busybox
            command: ["dd", "if=/dev/zero", "of=/dev/null"]
            name: main
            resoorces:
              limit: 1
              memory: 20Mi

При этом разрешается перегрузка общих ресурсов узла. Когда процесс пытается выделить память сверх своих лимитов - процесс уничтожается. Если после перезапуска ничего не меняется несколько раз, то выводится статус CrashLoopBackOff. 

Класс QoS модулей
^^^^^^^^^^^^^^^^^

Для того, чтобы сказать, какой из модулей удалять в случае большой нагрузки существует 3 класса качества обслуживания. BestEffort (самы низкий приоритет), Burstable, Guaranteed (самый высокий).

BestEffort - те модули, которые не содержат в своей конфигурации никаких лимитов. Для того, чтобы создать модуль с Guaranteed нужно использовать

- Запросы и лимиты как для процессора, так и для памяти

- Нужно установить это для каждого контейнера

- Они должны быть равными (Лимит = запросу).

Если у нас имеются процессы с одинаковым классом Qos то уничтожается тот, кто имеет большую оценку OOM. Эта оценка вычисляется из двух факторов: процента доступной памяти и фиксированной корректировки оценки OOM, которая основана на классе QoS модуля и запрошенной памяти контейенра.  


Установка стандартных запросов и лимитов для модулей в расчете на простанство имен. 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Если требуется устанавливать ресурсы для всех контейнеров, используют LimitRange. Пример создания LimitRange

.. code:: console

        apiVersion: v1
        kind: LimitRange
        metadata:
          name: example
        spec:
          limits:
          - type: Pod
            min:
              cpu: 50m
              memory: 5Mi
            max:
              cpu: 1
              memory: 1Gi
          - type: Container
            defaultRequest:
              cpu: 200m
              memory: 100Mi
            default:
              cpu: 200m
              memory: 100Mi
            min:
              cpu: 50m
              memory: 5Mi
            max:
              cpu: 1
              memory: 1Gi
            maxLimitRequestRatio
              cpu: 4
              memory: 10
          - type: PersistentVolumeClaim
            min:
              storage: 1Gi
            max:
              storage: 10Gi

Лимитирование общего объема ресурсов, доступных в пространстве имен
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Чтобы лимитировать пространства имен используется квота ресурсов ResourceQuota. Пример

.. code:: console

        apiVersion: v1
        kind: ResourceQuota
        metadata:
          name: cpu-and-mem
        spec:
          hard:
            requests.cpu: 400m
            requests.memory: 200Mi
            limits.cpu: 600m
            limits.memory: 500Mi
            requests.storage: 500Gi
            ssd.storageclass.storage.k8s.io/requests.storage: 300Gi
            standard.storageclass.k8s.io/request.storage.: 1Ti

Мониторинг потребления ресурсов модуля
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Запуск Heapster

.. code:: console

        $ minikube addons enable headpster

Вывод различной информации о поде или ноде

.. code:: console

        $ kubectl top node
        $ kubectl top pod

Для хранения всего используют InfluxDB - база данных временных рядов с открытым исходным кодом. 

Автоматическое масштабирование модулей и узлов кластера
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Создание hpa

.. code:: console

        apiVersion: extensions/v1beta1
        kind: Deployment
        metadata:
          name: kubia
        spec:
          replicas: 3
          template:
            metadata:
              name: kubia
              labels: 
                app: kubia
            spec:
              containers:
              - image: some/image
                name: main
                resources:
                  requests:
                    cpu: 100m

Для такого деплоймента можно создать hpa файл

.. code:: console

        apiVersion: autoscaling/v2beta1
        kind: HorizontalPodAutoScaler
        metadata:
          name: kubia
          ...

        spec:
          maxReplicas: 5
          metrics:
          - resources:
              name: cpu
              targetAverageUtilizationL 30
            type: Resource
          minReplicas: 1
          scaleTargetRef:
            apiVersion: extensions/v1beta1
            kind: Deployment
            name: kubia
        status:
          currentMetrics: []
          currentReplicas: 3
          desiredReplicas: 0

             
            
Error codes
^^^^^^^^^^^

137 - процесс был убит внешним сигналом
143 - по сути тоже
