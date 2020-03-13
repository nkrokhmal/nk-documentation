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
              
                


Error codes
^^^^^^^^^^^

137 - процесс был убит внешним сигналом
143 - по сути тоже
