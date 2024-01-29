## ELK Setting

### 디렉토리 구조

```
docker-elk						: ELK config 파일 및 Docker 관련 설정 Directory
├── elasticsearch				: Elasticsearch config 파일 및 Docker Image 설정 directory
│   ├── config
│   │   └── elasticsearch.yml	: Elasticsearch config 파일
│   └── Dockerfile 				: Elasticsearch Docker Image를 빌드하기 위한 파일
├── kibana 						: Kibana config 파일 및 Docker Image 설정 Directory
│   ├── config
│   │   └── kibana.yml` 		: Kibana config 파일
│   └── Dockerfile 				: Kibana Docker Image를 빌드하기 위한 파일
├── logstash 					: Logstash config 파일, pipeline, data, Docker Image 설정 Directory
│   ├── config
│   │   └── logstash.yml 		: Logstash config 파일
│   ├── pipeline 				: Logstash pipeline들을 정의하는 Directory
│   │   └── logstash.conf 		: Pipeline 정의 File
│   └── Dockerfile 				: Logstash Docker Image를 빌드하기 위한 파일
└── docker-compose.yml 			: ELK Docker Image를 빌드하고, container 설정을 정의하는 File
```

## 초기 설정

### 사용할 Docker 네트워크 생성

```sh
docker network create elk
```

### ELK 스택 실행

```sh
docker-compose up --build -d
```

### Elasticsearch 사용자 비밀번호 자동 설정 [ 수동 설정 auto -> interactive ]

```sh
docker exec -it $(docker ps -qf "name=elk-elasticsearch") sh bin/elasticsearch-setup-passwords auto
```

### 사용자 및 비밀번호 메모

```sh
# 예시 결과

pm_system
PASSWORD apm_system = TtwXd1UnwveywGySTbsk

Changed password for user kibana_system
PASSWORD kibana_system = TM4gAVh7aGwzovwucz5T

Changed password for user kibana
PASSWORD kibana = TM4gAVh7aGwzovwucz5T

Changed password for user logstash_system
PASSWORD logstash_system = Qj3DFRa9Bw8RSj5y3uJr

Changed password for user beats_system
PASSWORD beats_system = P69lIbip2cl5A7QBK71E

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = ZlYMLPCZYRALVDjtHZs7

Changed password for user elastic
PASSWORD elastic = mnqyhkiRCn1nXaW1wt0y
```

### 구성 파일 업데이트

- kibana 설정 ( kibana.yml )

```yml
# Kibana가 Elasticsearch에 연결할 때 사용할 사용자 ID를 지정합니다.
elasticsearch.username: kibana_system

# Kibana가 Elasticsearch에 연결할 때 사용할 사용자 PW를 지정합니다.
elasticsearch.password: <YOUR KIBANA PASSWORD>
```

- logstash 설정 ( logstash.yml )

```yml
# Logstash가 Elasticsearch에 모니터링 데이터를 전송할 때 사용할 사용자 ID를 지정합니다.
monitoring.elasticsearch.username: logstash_system

# Logstash가 Elasticsearch에 모니터링 데이터를 전송할 때 사용할 사용자 PW를 지정합니다.
monitoring.elasticsearch.password: <YOUR LOGSTASH PASSWORD>
```

- logstash 파이프라인 설정 ( logstash.conf )

```conf
output {
  elasticsearch {
    hosts 	=> [ 'http://elasticsearch:9200' ]
    user	=> 'logstash_system'
    password	=> '<YOUR LOGSTASH PASSWORD>'
    index 	=> '%{[@metadata][beat]}-%{[@metadata][version]}'
  }
}
```

### 서비스 재시작

Kibana와 Logstash에 변경사항을 적용하기 위해 서비스를 재시작합니다.

```sh
docker-compose up --build -d logstash kibana
```

### 접속 정보

```
Kibana: http://localhost:5601

ID: elastic
Password: <YOUR ELASTIC PASSWORD>
```

### 추가 설정

1. 좌측 상단 메뉴 탭 -> Management -> Stack Management -> Security -> Roles 생성

2. logstash role 생성

   - Cluster privileges: all

   - Index privileges - Indices: \*

   - Index privileges - Privileges: all

3. 좌측 상단 메뉴 탭 -> Management -> Stack Management -> Security -> Users 생성

4. logstash의 user 생성

   - Username: {유저 이름}

   - Password: {유저 비밀번호}

   - Privileges - Roles: {Role에서 설정한 Role Name 설정}

5. 생성한 user 정보로 logstash.yml, logstash.conf, kibana.yml 수정

- logstash 설정 ( logstash.yml )

```yml
# Logstash가 Elasticsearch에 모니터링 데이터를 전송할 때 사용할 사용자 ID를 지정합니다.
monitoring.elasticsearch.username: <YOUR LOGSTASH USER NAME>

# Logstash가 Elasticsearch에 모니터링 데이터를 전송할 때 사용할 사용자 PW를 지정합니다.
monitoring.elasticsearch.password: <YOUR LOGSTASH USER PASSWORD>
```

- logstash 파이프라인 설정 ( logstash.conf )

```conf
output {
  elasticsearch {
    hosts 	=> [ 'http://elasticsearch:9200' ]
    user	=> '<YOUR LOGSTASH USER NAME>'
    password	=> '<YOUR LOGSTASH USER PASSWORD>'
    index 	=> '%{[@metadata][beat]}-%{[@metadata][version]}'
  }
}
```

### 서비스 재시작

Logstash에 변경사항을 적용하기 위해 서비스를 재시작합니다.

```sh
docker-compose up --build -d logstash
```
