# k8s3
Kubernetes практика

Оркестрация - процесс автоматического развертывания и управления контейнерами
- Docker swarm (не хватает функий для сложных приложений)
- Kubernetes (самый популярный, не сложный и многофункциональный)
- Mesos (сложный, но продвинутый)

Архитектура

Noda эта машина физическая или виртуально на которой установлен кубернетес 

Вокер нода это компьютер на котором кубернетес будет запускать контейнеры, раньше они назывались миньонами

Кластер это набор сгруппированых вместе узлов, таким образом даже если один узел падает то наше приложение по-прежнему доступно для пользователей, более того наличие нескольких nod также помогают распределению нагрузки

Мастер это еще одна нода член кубернетес сконфигурированая как мастер. У нее особые функции наблюдать за состоянием других нод и быть ответственным за оркестрацию контейнеров на воркер нодах

Компоненты:

- API SERVER - frontend - единый интерфейс Kubernetes, все общаются с api server
- CONTAINER RUNTIME - среда выполнения контейнера - базовое ПО используется для запуска контейнеров
- CONTROLLER - мозг оркестрации, смотрят за состоянием нод, контейнеров, endpoint-ов и ответственны за реакцию за события на нодах. Принимают решения о создании новых контейнеров.
- SCHEDULER - ответственнен за распределение работ или контейнеров между нодами, ожидает появления контейнера, чтобы назначить его выполнение на ноду
- Служба KUBELET - агент который работает на каждом узле кластера, агент отвечает за то чтобы контейнеры работали на узлах должным образом
- Служба ETCD - хранилище ключ-значение, распределенная надежная БД используемая kubernetes для хранения информации нужной для управления кластера. Когда в кластере несколько докер нод, несколько мастеров ETCD хранит свою информацию на всех нодах кластера распределенным образом и отвечают за реализацию блокировок, не допуская конфликты между мастерами

Отличия worker и master

Worker нода или миньон это сервер где размещены контейнеры, например докер контейнер. Для запуска контейнеров докер в системе должна быть установлена среда выполнения контейнера в данном случае это докер, но вовсе не обязательно должен быть он есть и другие контейнер runtime и такие как rkt или cri-o 

Minikube - одноузловой кластер для тестов

https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/

Если установить kubectl перед миникуб то не нужно будет заниматься ее привязкой к кластеру миникуб это сделает за нас

https://kubernetes.io/ru/docs/tasks/tools/install-minikube/

Kubeadm необходимо
- Docker
- kubeadm
- Инициализация мастера
- POD Network
- Присоединение рабочих узолов к главному

**Простые абстракции**

Кубернетес не развертывает контейнеры непосредственно в ноды он инкапсулирует их в объекты которые называются поды

**Под** это отдельный экземпляр приложения, также под это самый маленький объект который ты можешь создать в кубернетес, самая маленькая деталь этого конструктора

В поде обычно один контейнер, в котором выполняются приложение, а чтобы отмасштабироваться вверх ты создаешь новые поды, а чтобы соскелиться вниз удаляешь существующие. Не стоит добавлять дополнительные контейнеры в существующие ноды для масштабирования приложения

В поде обычно один контейнер, но есть ли ограничение на запуск нескольких контейнеров в одном поде? Нет в одном поде может быть несколько контейнеров, но эти контейнеры не должны быть одного типа. В некоторых случаях требуется создать контейнер помощник который нес бы функции поддержки приложения из основного контейнера, такие как предпроцессинг введенных пользователем данных, обработка файлов загруженных пользователям или фиксация поведения приложения, главное требование для таких контейнеров, что время их жизни совпадает с продолжительностью работы основного контейнера с приложением. В этом случае оба контейнера буду частью одного пода и когда будет создан новый контейнер приложения, вместе будет создан помошник, а когда контейнер приложения умрет помощник тоже будет уничтожен вместе с подом.
```
kubectl run nginx --image nginx - ставим nginx
kubectl get pods - смотрим доступные поды
kubectl describe pod nginx - подробная информация
kubectl get pods -o wide - польная информация от get pods
```

**YAML для Kubernetes**

```yml
apiVersion: v1
kind: Pod
metadate:
  name: myapp-pod
  labels:
    app: myapp
    tier (type?): frontend
spec:
  containers:
  - name: nginx-container
    image: nginx
#  - name: busybox - как можно создать второй контейнер
#    image: busybox

```
```
kubectl create (apply) -f pod.yml - kubernetes создаст под
```

Манифест всегда содержит
- **apiVersion** - это версия api kubernetes который мы используем для создания объекта. В зависимости от того что мы пытаемся создать мы должны указать правильный и apiVersion (для pod - v1)
- **kind** - kind относится к типу объекта которые мы пытаемся создать в данном случае pod окей мы пишем под
- **metadata** - метаданные это данные об объекте такие  как его имя метки и так далее. В отличие от первых двух имеющих строковое значение это словари. Все что относится к метаданным будет являться дочерними элементами, то есть будут сдвинуты по правилам yml. Labels - словари. Type - для отслеживания
- **spec** - спецификация -  в зависимости от объекта который мы собираемся создать мы предоставляем kubernetes дополнительную информацию относящуюся к этому объекту. Она разная для разных типов объектов, поэтому важно запомнить или обратиться к разделу документации чтобы указать правильный формат. 

Это самый верхний или корневой уровень свойств. Все эти поля обязательны поэтому они должны присутствовать в файле конфигурации 
```
Kind  -  Version
Pod  -  v1
Service  -  v1
ReplicaSet - apps/v1
Deployment - apps/v1
```

**ReplicaSets**

**ReplicationController** (устарел, теперь используем ReplicaSet) - контроллер репликации помогает нам запускать несколько экземпляров под в кластере kubernetes тем самым обеспечивая высокую доступность. Контроллер репликации гарантируют что указанное количество исправных под будет постоянно работать - будь это всего лишь один под или сотня. Другая причина его использование распределение нагрузки между несколькими подами.
```yml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: frontend
spec:
  template: #Pod - берем из варианта с Pod
    metadate:
      name: myapp-pod
      labels:
        app: myapp
        tier (type?): frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3
```
```
kubectl create -f rcontroller.yml
kubectl get replicationcontroller
kubectl get pods
```
Важное отличие между ReplicationController и ReplicaSet - для ReplicaSet требуется определение селектора. Раздел селектор помогает ReplicaSet понять какие поды ему принадлежат. Потому что реплика сет также может управлять подами которые не были созданы как часть ReplicaSet
```yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: frontend
spec:
  template: #Pod - берем из варианта с Pod
    metadate:
      name: myapp-pod
      labels:
        app: myapp
        tier (type?): frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx
  replicas: 3
  selector:
    matchLabels:
      type: frontend
```
```
kubectl create -f rset.yml
kubectl get replicaset
kubectl get pods
```
Надо больше реплик - меняем replicas: 6 в rset.yml и запускаем repalce:
```
kubectl replace -f rset.yml - обновит набор реплик до требуемого значение 
```
или
```
kubectl scale --replicas=6 -f rset.yml
kubectl scale --replicas=6 -f myapp-rc - по имени! но не сохранит все в файл! при перезапуске будет запущено что было ранее
```
Редактирование в системе
```
kubectl edit rs myapp-rc - откроет копию в vi и после редакции проверит на ошибки, перепишет файл и применить (например запустит доп ноды)
```
Удаление
```
kubectl delete replicaset rset.yml
```

**Deployment**

После обновления важных компонетов, нам нужен процесс который обновит контейнеры один за другим, такое обновление называется rollin update. Предположим что одно из выполненных нами обновлений привело к непредвиденной ошибке и нас попросят отменить последнее действие, нам нужна возможность откатиться от недавно внесенных изменений, наконец если нам требуется внести изменения в свою среду например пропатчить ОС ноды или изменить распределение ресурсов вкупе с масштабированием или какие то другие вещи и мы не хотим применять каждое изменение сразу после исполнения команды, а предпочитаем поставить нагрузку на паузу и произвести требуемые изменения, а затем возобновить работу чтобы все изменения развертывались вместе все эти возможности доступны в **deployment**

Deployment выше в иерархии и обладает всем свойствами pod и replicaset
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: nginx
    tier: frontend
spec:
  selector:
    matchLabels:
      app: myapp
  replicas: 4
  template:
    metadate:
      name: nginx2
      labels:
        app: myapp
    spec:
      containers:
      - name: nginx
        image: nginx
```
```
kubectl create -f deploy.yml --record  - создание (--record - добавляет в history причину изменения)
kubectl get deployments - инфо
Deployment автоматически запустит replicaset поэтому можно запустить :
kubectl get replicaset
kubectl get pods
kubevtl get all - компнда покажет все deploy,rs,po
kubectl describe deployment myapp-deployment - выведет описание deployment
```
**Обновление и откат**

Каждый раз при создании нового deployment или обновления образа существующим запускается rollout - это процесс постепенного развертывания
или обновления контейнеров приложения. Rollout создает ревизию развертывания, его отпечаток snapshot назовем его revision 1. В будущем когда приложение будет обновлено то есть когда версия контейнера будет меняться на новую запустится новый rollout который создаст новую ревизию с именем revision 2. эта функция помогает нам отслеживать изменения внесенные в deployment и позволяет при необходимости вернуться к предыдущей развернутой версии.
```
kubectl rollout status deploy myapp-deployment - состояние выкатки
kubectl rollout history deploy myapp-deployment - просмотр списка ревизий и истории изменения
```
Стратегия Recreate - убиваем все контейнеры, потом поднимаем все контейнеры - есть время простоя

Rolling update - убиваем и поднимае по одному контейнеру (стандартная стратения deployment)

Разницв Recreate и Rolling update видна через (по созданию, убийству контейнеров)
```
kubectl describle deployments.apps myapp-deployment
```

**Обновление версии**
```
меняем в манифесте image: nginx: 1.7.1 дадее
kubectl apply -f deploy.yml

kubectl set image deployment/myapp-deployment nginx=nginx:1.7.1 - обновления образа, но в манифесте останется станое
```
Deployment при обновлении создает еще один replicaset2 и убивает поды в replicaset1, запускает их в replicaset2.

Если надо откатить изменения:
```
kubectl rollout undo deployment/myapp-deployment
```
**Команды:**
```
kubectl create -f deploy.yml - создание
kubectl get deployments - инфо
kubectl apply -f deploy.yml - принятие изменений
kubectl set image deployment/myapp-deployment nginx=nginx:1.7.1 - изменения
kubectl set deployments myapp-deployment --replicas=2 - изменения
kubectl rollout status deployment/myapp-deployment - состояние выкатки
kubectl rollout history deployment/myapp-deployment - просмотр списка ревизий и истории изменения
kubectl rollout undo deployment/myapp-deployment - откат
kubectl delete deploy myapp-deployment - удаление
```
**Сетевое взаимодействие**

В кубернетес ip адрес назначается поду (стандарнтые адреса  10.224.0.0) На одном узле все просто - кубернетес поднимает стандартную четь и поды могут через нее взаимодействовать.

Если на одном узле у нас несколько нод то не будет работать если узлы являются частью одного кластера, для подов назначены одинаковые ip это приведет конфликтом адресов в сети. когда мы занимаемся настройке кластера kubernetes автоматически не настраивает какие-либо сети чтобы решить эту нашу проблему. Собственно говоря kubernetes ожидает что мы сами настроим сеть удовлетворяющюю его базовым требованиям. Основные из этих требований заключаются в том что все контейнеры или поды в кластере kubernetes должны иметь возможность связываться друг с другом без необходимости настройки NAT, все ноды должны иметь возможность связываться с контейнерами и все контейнеры должны иметь возможность связываться с нодами в кластере.

Есть готовые решения - calico, flannel, cilium, weave, vmware nsx и другие

**Services (Службы)**

NodePort - когда служба делает доступным внутренни порт через порт на узле

ClustereIP - служба создает виртуальный ip адрес внутри кластера, чтобы обеспечить связь между различными службами, такими как набор фронтэндов или бекэндов

LoadBalancer - предоставляет балансировщик нагрузки для нашего сервиса, у поддерживаемых облачных провайдеров хорошим примером этого может быть распределение нагрузки между разными серверами в ярусе фронтэнда 

TargetPort - порт в поде (на него служба направляет запрос)

Port - порт службы

NodePort - порт на самой ноде (для внешнего доступа к веб серверам)
