## Laboratório AWS - Ingestão e Processamento de Dados Climáticos em Tempo Real (Tomorrow.io)

Este repositório demonstra uma arquitetura em nuvem para ingestão, processamento e notificação de dados climáticos em tempo real usando a API Tomorrow.io com serviços AWS.

---

### ✅ Pré-requisitos

1. Criar conta em: [Tomorrow.io](https://app.tomorrow.io/)
2. Gerar `TOMORROW_API_KEY`
   - Exemplo: `t5zlFP6aRbpswkTx5RQxwZoaMMFOlxTY`

---

### ⚛️ Etapas de Implementação

#### 1. Criar o Kinesis Data Stream
```bash
Nome: broker
```

#### 2. Criar IAM Role para Lambda Producer
```bash
Nome: producer_iam
Permissões:
  - AWSLambdaBasicExecutionRole
  - AmazonKinesisFullAccess
```

#### 3. Criar Lambda Function - Producer
```bash
Nome: producer
Runtime: Python 3.12
Role: producer_iam
Variável de ambiente: TOMORROW_API_KEY
Zip: lambda_function_producer.zip
```
- Realizar testes via aba *Test*.
- Verificar dados em Kinesis > aba Monitoring > Incoming data.

#### 4. Criar SNS (Simple Notification Service)
```bash
Nome: snsalerta
Tipo: Standard
ARN: arn:aws:sns:us-east-1:710271919494:snsalerta
```
- Adicionar E-mail e confirmar a inscrição.
- Adicionar telefone e confirmar via SMS.

#### 5. Criar IAM Role - Consumer Real-Time
```bash
Nome: consumerrealtime_iam
Permissões:
  - AmazonSNSFullAccess
  - AWSLambdaKinesisExecutionRole
```

#### 6. Criar Lambda - consumer_realtime
```bash
Nome: consumer_realtime
Runtime: Python 3.12
Role: consumerrealtime_iam
Variáveis de ambiente:
  - PRECIPITATION_PROBABILITY_THRESHOLD=70
  - WIND_SPEED_THRESHOLD=5
  - WIND_GUST_THRESHOLD=15
  - RAIN_INTENSITY_THRESHOLD=10
Trigger: Kinesis (broker)
```
- Testar com Lambda `producer`.

#### 7. Criar Agendamento no CloudWatch
```bash
Nome: producer_event
Tipo: Schedule (cron)
Cron: 0/3 * * * ? *
Lambda alvo: producer
```

#### 8. Criar Bucket S3
```bash
Nome: alertarealtime
Pastas:
  - raw/
  - gold/
```

#### 9. Criar IAM Role - Consumer Batch
```bash
Nome: consumerbatch_iam
Permissões:
  - AWSLambdaBasicExecutionRole
  - AmazonKinesisFullAccess
  - AmazonS3FullAccess
```

#### 10. Criar Lambda - consumer_batch
```bash
Nome: consumer_batch
Runtime: Python 3.12
Role: consumerbatch_iam
Variável: BUCKET_NAME=alertarealtime
Trigger: Kinesis (broker)
```
- Testar com Lambda `producer`. Verificar dados em S3/raw

#### 11. Criar IAM Role - Glue
```bash
Nome: etl_raw
Permissões:
  - AWSGlueServiceRole
  - AmazonS3FullAccess
  - CloudWatchLogsFullAccess
```

#### 12. Criar Crawler - raw_crawler
```bash
Fonte: s3://alertarealtime/raw
Banco de dados: raw_db
```

#### 13. Criar Glue Job - weather_job
```bash
Editor: Script (Spark)
Script: jobglue.py
Role: etl_raw
Output: s3://alertarealtime/gold/
```
- Executar o job e verificar dados em formato Parquet.

#### 14. Criar Crawler - gold_crawler
```bash
Fonte: s3://alertarealtime/gold
Banco de dados: weather
```

#### 15. Configurar Athena
```bash
Output: s3://alertarealtime/silver/
```
- Consultar as tabelas criadas com Glue.

---

Este laboratório é ideal para portfólios de Engenharia de Dados, demonstrando conhecimento prático com APIs, processamento em tempo real, monitoramento, integração com SNS e armazenamento otimizado.
