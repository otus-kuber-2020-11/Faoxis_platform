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
1) Разберитесь почему все pod в namespace kube-system восстановились после удаления
