# Debezium + Oracle 19 usando XStream + Event Hubs
 
Exemplo de utilização do Debezium + Oracle 19 usando XStream + Event Hubs

Pré-requisito: docker.

# Para executar

1) Crie um arquivo ".env" na raiz do diretório com as váriaveis de ambiente. Exemplo:
``` .env
DEBEZIUM_VERSION=1.8
EH_NAME=eventhub-dev
EH_CONNECTION_STRING=Endpoint=sb://eventhub-dev.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXX
PWD=C:/github/debezium-oracle-logminer
```
Ou então passe os valores ao executar o docker-compose

2) Acesse o site https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html para baixar o instant client da oracle e coloque o conteudo dele (extraido) na pasta -> debezium-with-oracle-jdbc -> oracle_instantclient.  
Por questões de licença esse driver não vem junto a imagem do debezium.  
O docker compose irá buildar uma imagem com o driver baixado manualmente na pasta -> debezium-with-oracle-jdbc -> oracle_instantclient.

3) Suba o compose
```
docker-compose up
```

4) Espere até que o oracle inicie e execute todos os scripts das pastas: ora-setup-scripts e ora-startup-scripts ... ... ... em torno de 10 min.

5) Cadastre o conector via API (https://debezium.io/documentation/reference/connectors/oracle.html), seguindo as instruções abaixo:

```
- Substituir os parametros antes de criar o conector. Exemplo usado: "Endpoint=sb://eventhub-dev.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXX", substitua pelo seu:

"database.history.kafka.bootstrap.servers": "eventhub-dev.servicebus.windows.net:9093",
"database.history.sasl.jaas.config": "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"$ConnectionString\" password=\"Endpoint=sb://eventhub-dev.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXX\";",
"database.history.producer.bootstrap.servers": "eventhub-dev.servicebus.windows.net:9093",
"database.history.producer.sasl.jaas.config": "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"$ConnectionString\" password=\"Endpoint=sb://eventhub-dev.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXX\";",
"database.history.consumer.bootstrap.servers": "eventhub-dev.servicebus.windows.net:9093",
"database.history.consumer.sasl.jaas.config": "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"$ConnectionString\" password=\"Endpoint=sb://eventhub-dev.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=XXX\";",

curl -i -X POST -H "Accept:application/json" -H  "Content-Type:application/json" http://localhost:8083/connectors/ -d @connector.json
```

6) Para conectar no banco use o usuário "debezium" e senha "dbz".  
O debezium irá usar o usuário "c##dbzuser" e senha "dbz" nas configurações do conector.

### Observações:
- Serão gerados 3 hubs/topicos dentro do Event Hubs respectivamente chamados de oracle-dev-configs, oracle-dev-offsets e oracle-dev-status. Eles podem ser configurados dentro do docker-compose alterando as váriaveis: CONFIG_STORAGE_TOPIC, OFFSET_STORAGE_TOPIC, STATUS_STORAGE_TOPIC.
- Esses topicos serão usados pelo Debezium para controlar o estado dos conectores cadastrados e o ponto em que a leitura foi realizada até o momento: offset.

### Conector verificar o status de um conector:
http://localhost:8083/connectors/oracle-connector  
http://localhost:8083/connectors/oracle-connector/status

# Para limpar o ambiente docker
```
docker-compose down
```

# Referências
https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-kafka-connect-tutorial  
https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/connect  
https://docs.microsoft.com/en-us/azure/event-hubs/event-hubs-kafka-connect-debezium  
https://debezium.io/documentation/reference/1.7/configuration/signalling.html  
https://www.youtube.com/watch?v=mzho5QS6CSk  
https://github.com/debezium/oracle-vagrant-box  
https://github.com/confluentinc/demo-scene/blob/master/oracle-and-kafka/docker-compose.yml  
https://github.com/Azure/azure-event-hubs-for-kafka/issues/61  
https://github.com/debezium/docker-images/blob/main/connect-base/1.8/Dockerfile  
https://medium.com/@lmramos.usa/debezium-cdc-oracle-931c8a62023b  
https://developers.redhat.com/blog/2021/04/19/capture-oracle-database-events-in-apache-kafka-with-debezium#setting_up_your_oracle_database  
https://github.com/confluentinc/demo-scene/tree/master/no-more-silos-oracle
