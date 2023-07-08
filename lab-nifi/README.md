# LAB NIFI
## Disclaimer
Esta configuração é puramente para fins de desenvolvimento local e estudos

## Pré-requisitos
- Docker
- Docker-Compose
- Terminal Linux
- Web browser
---
### Opcional
Parar todos os containers em execução
```
docker stop $(docker ps -a -q)
```
---

## Criando o ambiente Nifi com o docker compose
### Terminal do Linux
No diretório /lab-nifi execute o comando abaixo
```
docker-compose up -d
```
Verificando se os containers foram criados com sucesso

```
 docker container ls
```
Verificando as imagens que foram feitas download do docker-hub
```
 docker image ls
```
### Web browser
---
#### Opcional
Verificando se a ferramenta Kafdrop realizou o deploy com sucesso
http://localhost:19000/

>Kafdrop é uma interface de usuário da web simples, para visualizar tópicos Kafka e navegar em grupos de consumidores. A ferramenta exibe informações como corretores, tópicos, partições, consumidores e permite visualizar mensagens.

---

Acessar Nifi
```
https://localhost:8443/nifi
```
Usuário e senha
```
admin
PasswordNIFI!2023@
```
## Criar Hello World
### Web Browser
Criar Processor Group com o nome "Hello World"
- Arrastar o ícone, da barra de ferramentas superior, para o meio do canvas
  

Criar Processor GenerateFlowFile
- Arrastar o ícone, da barra de ferramentas superior, para o meio do canvas
- Em Custom Text adicionar "Hello World"

Criar Processor PutFile
- Setar propriedade Directory com
```
/tmp/hello
```
### No Terminal do Linux dentro do container lab-nifi_nifi_1
Abrir Terminal do container
```
docker exec -it lab-nifi_nifi_1 /bin/bash
```
```
cd ~
cd /tmp/hello
ls -ltr
```


## Criando tópico Kafka
### Terminal do Linux
```
docker exec -it lab-nifi_kafka_1 /bin/bash
```
```
kafka-topics --bootstrap-server localhost:9092 --topic alunos --create
```
```
kafka-topics --bootstrap-server localhost:9092 --topic professores --create
```

Listando os tópicos criados
```
kafka-topics --bootstrap-server localhost:9092 --list 
```

## Produzindo mensagens para o Kafka usando Nifi
### Web browser
Criar Processor Group com o nome "kafka-lab-publish"
- Arrastar o ícone, da barra de ferramentas superior, para o meio do canvas

Criar Processor GenerateFlowFile
- Arrastar o ícone, da barra de ferramentas superior, para o meio do canvas
- Em Custom Text adicionar
```
Nota,Nome
1,Fernando
1,Felipe
1,Yuri
2,Sandra
2,Beatriz
2,Lucas
3,Aureliano
3,Alexandre
3,Ernesto
3,Victor
3,Ronaldo
4,Teste
4,Fulano
4,Fulana
4,Ciclano
4,Ciclana
5,Beltrano
5,Beltrana
5,VemDisel
6,Thanos
6,VanDaime
7,Didi
7,Dede
8,Mussum
8,Zacarias
9,Pedro
9,Tony
9,Snow
10,Mulan
10,Fiona
```
Criar Processor UpdateAttribute
- Criar propriedade
    - Clicar no botão "+"
```
schema.name: test-schema
```

Criar Controller Service AvroSchemaRegistry
- No canvas raiz
    - Clicar com o botão direito e depois em Configure
- Clicar na aba Controller Services
- Clicar no botão "+"
- Selecionar AvroSchemaRegistry
- Clicar na engrenagem
- Clicar no botão "+"
- Criar propriedade
```
test-schema
```
- Valor da propriedade
```
{
   "type" : "record",
   "namespace" : "Test",
   "name" : "Alunos",
   "fields" : [
      { "name" : "Nota" , "type" : "int" },
      { "name" : "Nome" , "type" : "string" }
   ]
}
```

Criar Controller Service CSVReader
- Clicar no botão "+"
- Selecionar CSVReader
- Clicar na engrenagem
- Setar propriedade Schema Access Strategy com
```
Use 'Schema Name' Property
```
- Setar propriedade Schema Registry com
```
AvroSchemaRegistry
```
- Setar propriedade Value Separator com
```
,
```
- Set propriedade Treat First Line as Header com
```
true
```

Criar Controller Service AvroRecordSetWriter
- Clicar no botão "+"
- Selecionar AvroRecordSetWriter
- Clicar na engrenagem
- Setar propriedade Schema Access Strategy com
```
Use 'Schema Name' Property
```
- Setar propriedade Schema Registry com
```
AvroSchemaRegistry
```

Em Controller Services
- Clicar no no ícone de Raio para iniciar o serviço AvroSchemaRegistry 
- Clicar no no ícone de Raio para iniciar o serviço AvroRecordSetWriter
- Clicar no no ícone de Raio para iniciar o serviço CSVReader
- Voltar para o Processor Group kafka-lab-publish

Criar Processor PublishKafkaRecord_2_6
- Setar propriedade Kafka Brokers com
```
lab-nifi_kafka_1:29092
```
Setar propriedade Topic Name com
```
alunos
```
Setar propriedade Record Reader com
```
CSVReader
```
Setar propriedade Use Transactions com
```
false
```
Setar propriedade Record Writer com
```
AvroRecordSetWriter
```
Encerrar fluxo
- Marcar success terminate
- Marcar failure terminate

Conectar Processors
- Passar o mouse no meio do Processor e arrastar a seta para o próximo Processor


## Consumindo mensagens
### Terminal do Linux dentro do container lab-nifi_kafka1
```
kafka-console-consumer --bootstrap-server localhost:9092 --topic alunos
```

### Web Browser
Ligar Processors -> Play

## Consumindo mensagens do Kafka usando Nifi
### Web browser
Criar Processor Group com o nome "kafka-lab-consume"
- Arrastar o ícone, da barra de ferramentas superior, para o meio do canvas

Criar Processor ConsumeKafka_2_6
- Setar propriedade Kafka Brokers com
```
lab-nifi_kafka_1:29092
```
- Setar propriedade Topic Name com
```
professores
```
- Setar propriedade Record Reader com
```
CSVReader
```
- Setar propriedade Group ID com
```
test1
```
- Setar propriedade Honor Transactions com
```
false
```
- Setar propriedade Message Demarcator com
```
Empty string set
```

Criar Processor PutFile
- Setar propriedade Directory com
```
/tmp/datafolder
```

## Produzindo mensagens
### Terminal do Linux dentro do container lab-nifi_kafka1
```
kafka-console-producer --bootstrap-server localhost:9092 --topic professores
```
Copiar o valor abaixo e Colar no Producer
```
Nota,Nome
1,Fernando
1,Felipe
1,Yuri
2,Sandra
2,Beatriz
2,Lucas
3,Aureliano
3,Alexandre
3,Ernesto
3,Victor
3,Ronaldo
4,Teste
4,Fulano
4,Fulana
4,Ciclano
4,Ciclana
5,Beltrano
5,Beltrana
5,VemDisel
6,Thanos
6,VanDaime
7,Didi
7,Dede
8,Mussum
8,Zacarias
9,Pedro
9,Tony
9,Snow
10,Mulan
10,Fiona
```
### Em outro Terminal do Linux dentro do container lab-nifi_kafka1
```
docker exec -it lab-nifi_nifi_1 /bin/bash
cd ~
cd /tmp/datafolder
ls -ltr
```
