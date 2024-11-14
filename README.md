![image](https://github.com/user-attachments/assets/8ae57de6-2c7f-4c6f-99f9-e421d39214dc)
# Docker
## Начало
• Docker compose :smiley_cat:

1. ```git clone https://github.com/skl256/grafana_stack_for_docker.git``` 

нажимать "y"

2. ```cd grafana_stack_for_docker``` 

переход в папку, куда был склонирован репозиторий

3. ```sudo mkdir -p /mnt/common_volume/swarm/grafana/config``` 

 создание каталога для конфигурации Grafana внутри общей директории

4. ```sudo mkdir -p /mnt/common_volume/grafana/{grafana-config,grafana-data,prometheus-data,loki-data,promtail-data}```

cоздание нескольких директорий для хранения данных конфигурации и информации

5. ```sudo chown -R $(id -u):$(id -g) {/mnt/common_volume/swarm/grafana/config,/mnt/common_volume/grafana}```

изменение владельца этих директорий на текущего пользователя

6. ```touch /mnt/common_volume/grafana/grafana-config/grafana.ini```

создание пустого файла grafana.ini

7. ```cp config/* /mnt/common_volume/swarm/grafana/config/```

копирование всех файлов из локальной директории config в созданный каталог 

8. ```mv grafana.yaml docker-compose.yaml```

переименование файла grafana.yaml в docker-compose.yaml
