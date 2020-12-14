# Faoxis_platform
Faoxis Platform repository


## Домашняя работа 1. Введение в Kubernetes
### Порядок выполнения:
1) Установил `minikube`
2) Запусти `kubernetes` кластер командой `minikube start`
3) Создал [`Dockerfile`](kubernetes-intro/web/Dockerfile) с открытым портом `8000` и возможностью подгружать ресурсы из `/app/`.
4) Создал [манифест](kubernetes-intro/web-pod.yaml) с поднятием моего образа в `kubernetes` с `init` контейнером, который подкладывает файл `index.html` в папку с ресурсами.
5) Запустил под командой `kubectl apply -f web-pod.yaml`.
6) Убедился в работоспособности пода командой `kubectl describe pod web`.
7) Проверил, что скаченная страница подгружается командой `kubectl port-forward --address 0.0.0.0 pod/web 8000:8000`.
8) Получил файл с описанием нового сервис `frontend` командой `kubectl run frontend --image avtandilko/hipster-frontend:v0.0.1 --restart=Never --dry-run=client -o yaml > frontend-pod.yaml`.
9) Через команду `docker logs frontend` понял каких переменных окружения не хватает и добавил их в файле [`frontend-pod-health.yaml`](kubernetes-intro/frontend-pod-healthy.yaml).
### Ответы на вопросы:
#### Разберитесь почему все pod в namespace kube-system восстановились после удаления. Для ответа на вопрос попробуем проделать следующий действия:
1) Выполним команду `kubectl delete pod --all -n kube-system`, а затем `kubectl get pods -n kube-system`, чтобы убедиться, что поды на месте:
    ```shell
    NAME                               READY   STATUS    RESTARTS   AGE
    coredns-f9fd979d6-5d25r            1/1     Running   0          50s
    etcd-minikube                      1/1     Running   0          50s
    kube-apiserver-minikube            1/1     Running   0          50s
    kube-controller-manager-minikube   1/1     Running   0          50s
    kuяbe-proxy-nlw69                   1/1     Running   0          43s
    kube-scheduler-minikube            1/1     Running   0          49s
    ```
2) Далее нам необходимо проверить участие `Replica Set` и `Daemon Set` в поднятии подов. Для этого выполним следующие команды и посмотрим на результат:
    ```shell
    ➜  ~ kubectl get ds -n kube-system
    NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
    kube-proxy   1         1         1       1            1           kubernetes.io/os=linux   18m
    ➜  ~ kubectl get rs -n kube-system
    NAME                DESIRED   CURRENT   READY   AGE
    coredns-f9fd979d6   1         1         1       18m
    ```
3) По результату выше можно сделать вывод о том, что `kube-proxy` работает благодаря записи о нем в `Daemon Set`, а `coredns` с помощью `Replica Set`.
4) Остальные поды выглядят странно: у них нет динамического идентификатора, а значит принцип их работы отличается.
    После небольшого поиска в интернете, понял, что бывают динамические и статические поды. 
    Динамические поды поднимаются с помощью записи о них в `Replica Set` и `Daemon Set`.
    Статические поднимаются благодаря `kubelet` демону, который фиксирует статические поды по манифестам в директории `/etc/kubelet.d/` (по умолчанию) и "следит" за жизнью работающих статических подов.
    Сам `kubelet` востанавливает свою работу благодаря `linux` инструменту демонизации процессов `systemd`.



## Домашняя работа 2. Введение в контроллеры
### Порядок выполнения:
1) Создал кластер kind с [3 мастер нодами и 3-мя воркерами](kubernetes-controllers/kind-config.yaml)
2) Создал [frontend replicaset](kubernetes-controllers/frontend-replicaset.yaml) и задеблоил в `kind`
3) Попробовал обновить replicaset с новой версией и убедился, что не получилось с помощью команд 
   `kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'` (убедился, что реплика сет обновился) 
   и `kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'` (убедился, что версия подов не изменилась)
4) Обновление подов не происходит по той причине, что обновление образа в `replica set` не несет за собой 
5) На примере [paymentservice-deployment.yaml](kubernetes-controllers/paymentservice-deployment.yaml) удалось убедиться,
    что сущность `Deployment` решает проблему.
6) Кроме того, на примере образа `node-exporter` понял как работает [`DaemonSet`](kubernetes-controllers/node-exporter-daemonset.yaml),
    что позволяет очень просто настроить мониторинг всех нод, включая `master` ноды.
7) В процессе выполнения были сделаны все дополнительные задания. Было очень интересно.
