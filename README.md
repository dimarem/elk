# ELK (Elasticsearch, Kibana, Logstash)

Пример создания HA-кластера ELK в одной зоне доступности.

> **Важно:**
>
> - для data-узлов elasticsearch для установки максимального значения используемой памяти (mmap) используется init-контейнер, что требует возможность запуска привелигированных контейнеров, в ином случае следует использовать DaemonSet или ComputeClass

## Деплой

1. Установка CRD (Custom Resource Definitions):

```bash
kubectl create -f https://download.elastic.co/downloads/eck/3.3.1/crds.yaml
```

2. Установка оператора и RBAC:

```bash
kubectl apply -f https://download.elastic.co/downloads/eck/3.3.1/operator.yaml
```

3. Мониторинг установки оператора:

```bash
kubectl -n elastic-system logs -f statefulset.apps/elastic-operator
```

4. Проверка статуса оператора:

```bash
kubectl get -n elastic-system pods
```

5. Создать группу ВМ из трех узлов с 4 ядрами и 32 Гб ОЗУ, тип диска - SSD.

6. Применить манифест:

```bash
helm upgrade --install --rollback-on-failure elk elk-chart --namespace elastic-system
```

## Информация по кластеру

1. Вывести статус кластера:

```bash
kubectl get elasticsearch -n elastic-system
```

2. Информация по ресурсам кластера:

```bash
kubectl get all -n elastic-system
```

3. Информация по PV и PVC:

```bash
kubectl get pv,pvc -n elastic-system
```

## Обращение к Elasticsearch API

1. Пробросить порт:

```bash
kubectl port-forward service/elk-elasticsearch-es-http 9200 -n elastic-system
```

2. Получить учетные данные:

```bash
PASSWORD=$(kubectl get secret elk-elasticsearch-es-elastic-user -n elastic-system -o go-template='{{.data.elastic | base64decode}}')
```

3. Выполнить запрос:

```bash
curl -u "elastic:$PASSWORD" -k "https://localhost:9200"
# {
#   "name" : "elk-es-master-0",
#   "cluster_name" : "elk",
#   ...
#   },
#   "tagline" : "You Know, for Search"
# }
```

## Подключение к Kibana

1. Получить пароль:

```bash
kubectl get secret elk-elasticsearch-es-elastic-user -n elastic-system -o go-template='{{.data.elastic | base64decode}}'
```

2. Получить ip адрес:

```bash
kubectl get svc -n elastic-system
# NAME                                 TYPE           CLUSTER-IP      EXTERNAL-IP       PORT(S)          AGE
# elk-kibana-kb-http                   LoadBalancer   10.96.226.120   158.160.216.128   5601:31484/TCP   5m52s
```

3. Подключиться к UI по адресу `https://<EXTERNAL-IP>:5601`, введя полученный пароль и имя пользователя `elastic`.

<img width="1919" height="914" alt="dashboard" src="https://github.com/user-attachments/assets/3ceb5d33-0bd1-4996-b368-9c315db611ff" />


## Удаление

```bash
helm uninstall elk --namespace elastic-system
```
