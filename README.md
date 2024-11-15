![image](https://github.com/user-attachments/assets/8ae57de6-2c7f-4c6f-99f9-e421d39214dc)
# Docker
## Начало
• Docker compose :smiley_cat:

1. ```git clone https://github.com/skl256/grafana_stack_for_docker.git``` 

![image](https://github.com/user-attachments/assets/3b048135-ef79-42ca-9fff-f623ae73e245)

нажимать "y"

2. ```cd grafana_stack_for_docker``` 

переход в папку, куда был склонирован репозиторий

3. ```sudo mkdir -p /mnt/common_volume/swarm/grafana/config``` 

 создание каталога для конфигурации Grafana внутри общей директории

4. ```sudo mkdir -p /mnt/common_volume/grafana/{grafana-config,grafana-data,prometheus-data,loki-data,promtail-data}```

![image](https://github.com/user-attachments/assets/d2b8471f-ef1d-4aba-9e73-c46b3d8605a2)

cоздание нескольких директорий для хранения данных конфигурации и информации

5. ```sudo chown -R $(id -u):$(id -g) {/mnt/common_volume/swarm/grafana/config,/mnt/common_volume/grafana}```

изменение владельца этих директорий на текущего пользователя

6. ```touch /mnt/common_volume/grafana/grafana-config/grafana.ini```

создание пустого файла grafana.ini

7. ```cp config/* /mnt/common_volume/swarm/grafana/config/```

копирование всех файлов из локальной директории config в созданный каталог 

8. ```mv grafana.yaml docker-compose.yaml```

переименование файла grafana.yaml в docker-compose.yaml

## Установка последней версии и утилиты docker-compose
1. ```sudo yum install curl```

![image](https://github.com/user-attachments/assets/de4c27a9-6e24-400f-9cce-413f3a9bec3a)

2. ```COMVER=$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep 'tag_name' | cut -d" -f4)```

![image](https://github.com/user-attachments/assets/823988eb-99b3-4166-9a30-a1635203bdf8)

3. ```sudo curl -L "https://github.com/docker/compose/releases/download/$COMVER/docker-compose-$(uname -s)-$(uname -m)" -o /usr/bin/docker-compose```

4. ```sudo chmod +x /usr/bin/docker-compose```

![image](https://github.com/user-attachments/assets/3c43cc6c-bfc5-473d-82f2-25f1e47b1e33)


5. ```docker-compose --version```

## Установка и настройка Docker

1.```sudo wget -P /etc/yum.repos.d/ https://download.docker.com/linux/centos/docker-ce.repo```

![image](https://github.com/user-attachments/assets/b941d273-bed6-4c3d-b0b3-cd3f67913762)

2.```sudo yum install docker-ce docker-ce```

3.```sudo systemctl enable docker --now```

4.```docker compose up -d```

![image](https://github.com/user-attachments/assets/571c550b-c9b0-49ea-9368-b5fd61843735)


# Конфигурация отдельных сервисов

## Prometheus

После установки Docker:

```sudo vi docker-compose.yaml``` (Должно быть расширение .yaml)

## Далее, в текстовом редакторе нужно поставить node-exporter после services.

• Для изменения в текстовом редакторе, нужно нажать insert на клавиатуре

• Чтобы сохранить что-то в текстовом редакторе, нужно нажать клавишу Esc и написать :wq! 

```
 node-exporter:
    image: prom/node-exporter
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    container_name: exporter < запомнить название name
    hostname: exporter
    command:
      - --path.procfs=/host/proc
      - --path.sysfs=/host/sys
      - --collector.filesystem.ignored-mount-points
      - ^/(sys|proc|dev|host|etc|rootfs/var/lib/docker/containers|rootfs/var/lib/docker/overlay2|rootfs/run/docker/netns|rootfs/var/lib/docker/aufs)($$|/)
    ports:
      - 9100:9100
    restart: unless-stopped
    environment:
      TZ: "Europe/Moscow"
    networks:
      - default
```

```
cd
cd /mnt/common_volume/swarm/grafana/config 
sudo vi prometheus.yaml 
```
• В данном файле нужно изменить IP адрес, затем вставить в targets, а после двоеточий цифры оставить 


# Grafana

1. Перейти на сайт ```localhost:3000```

2. Имя пользователя: ```admin```

   Пароль: ```admin```

• Код Grafana: 3000

• Код Prometheus: http://prometheus:9090

3. В меню выбрать вкладку Dashboards и создать Dashboard

• Нажать кнопку +Add visualization, затем "Configure a new data source"

• Выбрать Prometheus

4. Connection

```http://prometheus:9090```

5. Authentication

Basic authentication

• Имя пользователя: ```admin```

• Пароль:```admin```

Нажать на Save & test и должно показывать зелёную галочку

6. В меню выбрать вкладку Dashboards и создать Dashboard

• Нажать кнопку "Import dashboard"

• ```Find and import dashboards for common applications at grafana.com/dashboards: 1860```

Нажать кнопку ```Load```

• Select Prometheus, нажать кнопку ```Import```

![image](https://github.com/user-attachments/assets/9e7cc280-b613-437f-b076-01f935f59db7)

## Если не работает, то перейти

```
cd grafana_stack_for_docker
sudo docker compose stop
sudo docker compose up -d
```

Перезапуск докера.

# VicroriaMetrics

• Изменить docker-compose.yaml:

```
cd grafana_stack_for_docker
sudo vi docker-compose.yaml
```

В самом текстовом редакторе после ```prometheus``` вставить:

```
  vmagent:
    container_name: vmagent
    image: victoriametrics/vmagent:v1.105.0
    depends_on:
      - "victoriametrics"
    ports:
      - 8429:8429
    volumes:
      - vmagentdata:/vmagentdata
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - "--promscrape.config=/etc/prometheus/prometheus.yml"
      - "--remoteWrite.url=http://victoriametrics:8428/api/v1/write"
    restart: always
  # VictoriaMetrics instance, a single process responsible for
  # storing metrics and serve read requests.
  victoriametrics:
    container_name: victoriametrics
    image: victoriametrics/victoria-metrics:v1.105.0
    ports:
      - 8428:8428
      - 8089:8089
      - 8089:8089/udp
      - 2003:2003
      - 2003:2003/udp
      - 4242:4242
    volumes:
      - vmdata:/storage
    command:
      - "--storageDataPath=/storage"
      - "--graphiteListenAddr=:2003"
      - "--opentsdbListenAddr=:4242"
      - "--httpListenAddr=:8428"
      - "--influxListenAddr=:8089"
      - "--vmalert.proxyURL=http://vmalert:8880"
    restart: always
```

Сохранить и выйти

## Затем, зайти в ```connection```

• там, где писали ```http:prometheus:9090``` , написать  ```http:victoriametrics:9090``` и заменить имя из ```Prometheus-2``` в ```Vika```

• нажать на ```dashboards add visualition``` выбрать ```Vika```

• снизу изменить на ```cod```

 Перейти в терминал и написать:

```
echo -e "# TYPE OILCOINT_metric1 gauge\nOILCOINT_metric1 0" | curl --data-binary @- http://localhost:8428/api/v1/import/prometheus  
curl -G 'http://localhost:8428/api/v1/query' --data-urlencode 'query=OILCOINT_metric1'
```
 (Значение 0 изменить на любое другое)

![image](https://github.com/user-attachments/assets/c5cc8a2f-6c7e-4e54-9535-e657e3674d07)


• Скопировать переменную ```OILCOINT_metric1``` и вставить в ```cod```

• Нажать ```run```

![image](https://github.com/user-attachments/assets/9e55b942-289c-469e-949b-eab10d2efb09)







