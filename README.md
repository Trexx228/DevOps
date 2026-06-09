# DevOps
Настройки сервера под мониторинг

ПОДГОТОВКА ОС
1. Обновление системы: sudo apt update && sudo apt upgrade -y
2. Установка базового софта: sudo apt install -y curl git htop net-tools unzip
3. Проверка интернета: curl -I https://google.com
4. Проверка установленных утилит: git --version; curl --version


УСТАНОВКА DOCKER
1. Подготовка и установка: sudo apt update && sudo apt install -y ca-certificates curl gnupg lsb-release && sudo install -m 0755 -d /etc/apt/keyrings && curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg && sudo chmod a+r /etc/apt/keyrings/docker.gpg && echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null && sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin && sudo usermod -aG docker $USER

sudo reboot

2. Проверка: docker --version && docker compose version && docker run hello-world


СОЗДАНИЕ СТРУКТУРЫ ПРОЕКТА И БАЗОВЫХ КОНФИГОВ
1. Создание директории и переход в неё: mkdir -p ~/soc-lab && cd ~/soc-lab
2. Создание docker-compose.yml
3. Создание Dockerfile.logstash
4. Создание logstash.conf
5. Проверка созданных файлов: ls -la


ЗАПУСК СТЕКА И ПРОВЕРКА РАБОТОСПОСОБНОСТИ
1. Запуск стека: cd ~/soc-lab && sudo docker compose up -d
2. Проверка статуса контейнеров (Все контейнеры должны быть в статусе running): sudo docker compose ps
3. Проверка логов OpenSearch (ждем 30-60 сек после старта): sudo docker logs opensearch --tail 20
4. Тест OpenSearch API (ловим JSON с версией 2.14.0): curl http://localhost:9200
5. Тест OpenSearch Dashboards (ждём HTTP/1.1 200 OK или 302 Found): curl -I http://localhost:5601
6. Проверка, что Logstash собирает логи: sudo docker logs logstash --tail 10
7. Проверка индексов в OpenSearch (ждём индексы logstash-lab-2026.06.xx): curl "http://localhost:9200/_cat/indices?v"



НАСТРОЙКА ILM
1. Создание файла политики ILM: echo '{"policy":{"description":"Lab policy","default_state":"hot","states":[{"name":"hot","actions":[],"transitions":[{"state_name":"delete","conditions":{"min_index_age":"7d"}}]},{"name":"delete","actions":[{"delete":{}}]}]}}' > policy.json
2. Создание политики в OpenSearch (ждём "_id": "logstash-lab-policy"): curl -X PUT "http://localhost:9200/_plugins/_ism/policies/logstash-lab-policy" -H "Content-Type: application/json" -d @policy.json
3. Создание шаблона индекса (ждём {"acknowledged":true}): curl -X PUT "http://localhost:9200/_index_template/logstash-lab-template" -H "Content-Type: application/json" -d '{"index_patterns":["logstash-lab-*"],"template":{"settings":{"number_of_replicas":0,"opendistro.index_state_management.policy_id":"logstash-lab-policy"}}}'
4. Получение списка индексов (ищем имя индекса logstash-lab, типа logstash-lab-2026.06.09): curl "http://localhost:9200/_cat/indices?v"
5. Привязка политики к существующему индексу (типа logstash-lab-2026.06.09)
6. Проверка статуса ILM (ищем "policy_id": "logstash-lab-policy"): curl "http://localhost:9200/_plugins/_ism/explain/logstash-lab-2026.06.09"
