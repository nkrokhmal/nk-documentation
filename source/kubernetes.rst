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

Команда для выолнения curl запроса изнутри какой-то поды. В этой команде (--) используется для сигнала kubectl об окончании командных параметров

.. code:: console

        $ kubectl exec podname -- curl -s http://ip

Команда на получение всех endpoints для службы

.. code:: console

        $ kubectl get endpoints kubia



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

Информация о пода с учетом определенных меток

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

Error codes
^^^^^^^^^^^

137 - процесс был убит внешним сигналом
143 - по сути тоже
