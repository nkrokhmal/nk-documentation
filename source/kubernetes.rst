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




          

Error codes
^^^^^^^^^^^

137 - процесс был убит внешним сигналом
143 - по сути тоже
