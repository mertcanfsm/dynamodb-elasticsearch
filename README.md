# AWS DynamoDB - Elasticsearch pipeline

## Elasticsearch kurulum

Java 8 runtime kurun, eğer yoksa:

```bash
sudo yum install java-1.8.0-openjdk
```

Elasticsearch 2.3.5 indirin:

```bash
wget https://download.elastic.co/elasticsearch/release/org/elasticsearch/distribution/rpm/elasticsearch/2.3.5/elasticsearch-2.3.5.rpm
```
İndirdiğinizi kurun:

```bash
sudo rpm --install elasticsearch-2.3.5.rpm
```
**Not:** Daha yüksek Elasticsearch sürümleri kullanacağımız plugin ile çalışmayabilir.

Elasticsearch'i root olarak çalıştıramazsınız. Root olmadan çalıştırmak için gerekli izinleri verin:

```bash
sudo chown elasticsearch:elasticsearch -R /usr/share/elasticsearch
sudo chown elasticsearch:elasticsearch -R /var/log/elasticsearch
sudo chown elasticsearch:elasticsearch -R /var/lib/elasticsearch
sudo chown elasticsearch:elasticsearch -R /etc/elasticsearch
```
Arkaplan program olarak çalıştırmak için aşağıdaki komutları girin.

```bash
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
sudo systemctl start elasticsearch.service
```
Elasticsearch çalıştığından emin olmak için aşağıdaki komutu çalıştırın. Eğer json formatında çıktı geliyorsa, Elasticsearch çalışıyor demek.

```bash
curl localhost:9200
```
## Logstash kurulum

Docker engine kurun:

```bash
sudo amazon-linux-extras install docker
```

Docker arkaplan olarak çalıştırın:

```bash
sudo systemctl daemon-reload
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
sudo systemctl start docker
```

Logstash-dynamodb-streams image pull edin:

```bash
sudo docker pull mantika/logstash-dynamodb-streams
```

Logstash arkaplan çalıştırmak için aşağıdaki komutu girin.

`aws_access_key_id`, `aws_secret_access_key` ve `table_name` kısımlarını doldurun.

**Not:** Sadece us-east-1 bölgesi DynamoDB tabloları çalışmaktadır.

**Not:** Birden fazla tablo için `input` içerisindeki `dynamodb` kısmını duplicate edip `table_name` değiştirin.

**Not:** DynamoDB tabloları streaming için açık olmalı ve `New and old images` seçeneği seçilmiş olmalı.

```bash
sudo docker run -d --net="host" mantika/logstash-dynamodb-streams -e '
input { 
    dynamodb{endpoint => "dynamodb.us-east-1.amazonaws.com" 
    streams_endpoint => "streams.dynamodb.us-east-1.amazonaws.com" 
    view_type => "new_and_old_images" 
    aws_access_key_id => "" 
    aws_secret_access_key => "" 
    table_name => ""} 
} 
filter {
    dynamodb {}
}
output { 
    elasticsearch 
        { hosts => ["localhost"] } 
}'
```
Verilerin DynamoDB'den Elasticsearch'e gelmesi için biraz bekleyin.

Aşağıdaki komutu çalıştırın:

```bash
curl localhost:9200/_aliases
```
Eğer aşağdaki gibi bir çıktı geliyorsa, pipeline hazır demektir. Artık Elasticsearch ile arama yapabilirsiniz.

```bash
{"logstash-2021.08.24":{"aliases":{}}}
```

## Durdurma

Elasticsearch, Logstash ve Docker kapatmak için aşağıdaki komutları girin:

```bash
sudo systemctl stop elasticsearch.service
sudo systemctl stop docker
```

## Dokümantasyon

[Elasticsearch 2.3 Docs](https://www.elastic.co/guide/en/elasticsearch/reference/2.3/index.html)

[Logstash 2.3 Docs](https://www.elastic.co/guide/en/logstash/2.3/index.html)

Daha fazla Logstash ayarı için [buraya tıklayın](https://github.com/awslabs/logstash-input-dynamodb#setting-description).
