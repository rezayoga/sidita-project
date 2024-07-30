# sidita-project

### SIDITA v3
Aplikasi SIDITA v3 dibangun menggunakan arsitektur microservices yang memisahkan berbagai fungsi aplikasi ke dalam beberapa layanan yang kecil dan independen. SIDITA v3 terdiri dari 5 services, yaitu:
- Front-end → antarmuka user
- Auth → Bertugas untuk menghandle autentikasi, registrasi, dan pengaturan master data berkaitan dengan role, organisasi, dan contractor
- Dashboard → Bertugas untuk menghandle statistik data
- Project → Core Bisnis Logic SIDITA v3, pengelolaan proyek, kontrak, termyn, progres laporan dan berita acara
- Logger → Bertugas menyimpan log catatan aktivitas user

Setiap layanan memiliki tanggung jawab yang spesifik dan berkomunikasi satu sama lain melalui protocol HTTP dan GRpc. Berikut adalah komponen yang digunakan dalam SIDITA v3:

#### SIDITA v3 menggunakan 5 - 6 Machine (VM), berikut adalah detail VM yang digunakan dan sub komponen aplikasinya:
	 1. Group - Load Balancer
		- Server 1:
			- HaProxy/NGINX
	 2. Group - Monitoring
		- Server 2:
			- Grafana
			- Prometheus
	 3. Group - App
		- Server 3
			- Containerd
			- K8s
			- Containered FE App
			- Containered BE App
		- Server 4
			- Celery App
	 4. Group - Storage	
		- Server 5
			- PostgreSQL
			- Redis
			- MongoDB
			- SeaweedFS
			- RabbitMQ

#### Stack Used
![enter image description here](https://i.ibb.co.com/qsSFFG3/Screenshot-2024-07-17-at-10-33-29.png)

#### Arsitektur dan Alur Data SIDITA v3
![enter image description here](https://i.ibb.co.com/Q8FSmgr/Architecture-Document.jpg)

#### Deployments
Setiap service pada SIDITA v3 memiliki environment variable yang harus dikonfigurasi pada file konfigurasi K8s yang akan digunakan untuk menyimpan seluruh variable yang akan digunakan oleh aplikasi. 

Services SIDITA v3 hosted pada repository Docker, berikut registry repository SIDITA v3:
- Frontend Service:
    - dptpusmanpro/nextjs-sidita
 - Service Auth:
   	- dptpusmanpro/auth-service 	
 - Dashboard Service:
   	- dptpusmanpro/dashboard-service 	
 - Project Service:
   	- dptpusmanpro/project-service 	
  - Logger Service:
    - dptpusmanpro/logger-service

Deployments Steps:
1. Server and VMs Setup
2. Starting standalone storage app, PostgreSQL, Redis, MongoDB, RabbitMQ, SeaweedFS
3. Populate PostgreSQL database, schema, metadata creation, initial data insertion
4. Persiapan K8s Cluster
5. Persiapan Kubernetes Deployment dan Service Files, penyesuaian environment
6. LoadBalancer & Proxy setup
7. Konfigurasi propagasi Naming Service
8. Start Deploy!

#### Docker Compose Configuration File dan Environment Variables
```yaml
version: '3'

services:
  auth-service:
    image: dptpusmanpro/auth-service:0.0.79
    ports:
      - "8080:8080"
    deploy:
      mode: replicated
      replicas: 1
    environment:
      JWT_ISSUER: "PUSMANPRO PLN"
      JWT_AUDIENCE: "api.rc-sidita.my.id"
      COOKIE_DOMAIN: ".rc-sidita.my.id"
      COOKIE_NAME: "__Secure-siditav3"
      COOKIE_SECURE: "true"
      # COOKIE_NAME: "sidita"
      # COOKIE_SECURE: "false"
      APP_DOMAIN: "api.rc-sidita.my.id"
      LOGGER_SERVICE_PORT: "5001"
      LOGGER_SERVICE_HOST: logger-service
      DSN: "host=10.66.66.3 port=5432 user=postgres password=51DiT4_POst6R3sQL dbname=sidita-v3 sslmode=disable timezone=UTC+7 connect_timeout=5"
      # DSN: /run/secrets/dsn
      REDIS_DSN: "redis://:vDBAAwyqiEz4%2F9OupVhu59hrRk3VT4SqsrMDVKqp8E3NTv3vUJCSX4lIs8t9tcGT7rV666OFPEFZGKFw@10.66.66.3:6379/0"
      JWT_SECRET: "7E0eH-Jck1Dc2pKRdrZwEUTWbYKf2-S4rhKil_dalHF-ht2lCnrDHufZUng037tY7PNJNf_UEVd8Qqb1GgHS-lzEPK-N0pTWHMZj5U36ODeC7n13vLU0IWLh46xnkc1WWghWJz1oh4j87mNzmQjDlaQodoICaoOxglY1xbr9moE"
      SEAWEEDFS_FILER: "http://10.66.66.3:8888"
      BASEURL: "https://api.rc-sidita.my.id"
      MONGO_USER: "adminUser"
      MONGO_PASSWORD: "ZXgwB6pHmh0hVPLKAjZECSCBd9yucdTILxohUQ7QoGHrt7DwDp"
      MONGO_DB: "sidita"
      MONGO_DSN: "mongodb://adminUser:ZXgwB6pHmh0hVPLKAjZECSCBd9yucdTILxohUQ7QoGHrt7DwDp@103.175.221.64:27017/sidita?directConnection=true"
      MONGO_DB_NAME: "sidita"
      MONGO_COLLECTION_NAME: "logs"
      IAM_CLIENT_ID: "PvYW2FxACt5IXqSQSvbY"
      IAM_CLIENT_SECRET: "FQJvYFhUJWrl@EPocbPDZy2eTMkYuJ"
      IAM_AUTH_URL: "https://iam.pln.co.id/svc-core/oauth2/auth"
      IAM_TOKEN_URL: "https://iam.pln.co.id/svc-core/oauth2/token"
      IAM_ISSUER: "https://iam.pln.co.id/svc-core/oauth2"
      IAM_HR_INFO_URL: "https://iam.pln.co.id/svc-core/account/hrinfo"
      IAM_LOGOUT_URL: "https://iam.pln.co.id/svc-core/oauth2/session/end"

  logger-service:
    image: dptpusmanpro/logger-service:0.0.20
    deploy:
      mode: replicated
      replicas: 1
    environment:
      GRPC_PORT: "5001"
      MONGO_USER: "adminUser"
      MONGO_PASSWORD: "ZXgwB6pHmh0hVPLKAjZECSCBd9yucdTILxohUQ7QoGHrt7DwDp"
      MONGO_DB: "sidita"
      MONGO_DSN: "mongodb://adminUser:ZXgwB6pHmh0hVPLKAjZECSCBd9yucdTILxohUQ7QoGHrt7DwDp@10.66.66.3:27017/sidita?directConnection=true"#

  project-service:
    image: dptpusmanpro/project-service:0.0.111
    ports:
      - "8081:8081"
    deploy:
      mode: replicated
      replicas: 1
    environment:
      JWT_ISSUER: "PUSMANPRO PLN"
      JWT_AUDIENCE: "api.rc-sidita.my.id"
      COOKIE_DOMAIN: ".rc-sidita.my.id"
      COOKIE_NAME: "__Secure-siditav3"
      COOKIE_SECURE: "true"
      # COOKIE_NAME: "sidita"
      # COOKIE_SECURE: "false"
      APP_DOMAIN: "api.rc-sidita.my.id"
      LOGGER_SERVICE_PORT: "5001"
      LOGGER_SERVICE_HOST: logger-service
      DSN: "host=10.66.66.3 port=5432 user=postgres password=51DiT4_POst6R3sQL dbname=sidita-v3 sslmode=disable timezone=UTC+7 connect_timeout=5"
      REDIS_DSN: "redis://:vDBAAwyqiEz4%2F9OupVhu59hrRk3VT4SqsrMDVKqp8E3NTv3vUJCSX4lIs8t9tcGT7rV666OFPEFZGKFw@10.66.66.3:6379/0"
      JWT_SECRET: "7E0eH-Jck1Dc2pKRdrZwEUTWbYKf2-S4rhKil_dalHF-ht2lCnrDHufZUng037tY7PNJNf_UEVd8Qqb1GgHS-lzEPK-N0pTWHMZj5U36ODeC7n13vLU0IWLh46xnkc1WWghWJz1oh4j87mNzmQjDlaQodoICaoOxglY1xbr9moE"
      SEAWEEDFS_FILER: "http://10.66.66.3:8888"

  dashboard-service:
    image: dptpusmanpro/dashboard-service:0.0.1
    ports:
      - "8083:8083"
    deploy:
      mode: replicated
      replicas: 1
    environment:
      JWT_ISSUER: "PUSMANPRO PLN"
      JWT_AUDIENCE: "api.rc-sidita.my.id"
      COOKIE_DOMAIN: ".rc-sidita.my.id"
      COOKIE_NAME: "__Secure-siditav3"
      COOKIE_SECURE: "true"
      # COOKIE_NAME: "sidita"
      # COOKIE_SECURE: "false"
      APP_DOMAIN: "api.rc-sidita.my.id"
      LOGGER_SERVICE_PORT: "5001"
      LOGGER_SERVICE_HOST: logger-service
      DSN: "host=10.66.66.3 port=5432 user=postgres password=51DiT4_POst6R3sQL dbname=sidita-v3 sslmode=disable timezone=UTC+7 connect_timeout=5"
      # DSN: /run/secrets/dsn
      REDIS_DSN: "redis://:vDBAAwyqiEz4%2F9OupVhu59hrRk3VT4SqsrMDVKqp8E3NTv3vUJCSX4lIs8t9tcGT7rV666OFPEFZGKFw@10.66.66.3:6379/0"
      JWT_SECRET: "7E0eH-Jck1Dc2pKRdrZwEUTWbYKf2-S4rhKil_dalHF-ht2lCnrDHufZUng037tY7PNJNf_UEVd8Qqb1GgHS-lzEPK-N0pTWHMZj5U36ODeC7n13vLU0IWLh46xnkc1WWghWJz1oh4j87mNzmQjDlaQodoICaoOxglY1xbr9moE"
      SEAWEEDFS_FILER: "http://10.66.66.3:8888"
      BASEURL: "https://api.rc-sidita.my.id"
      MONGO_USER: "adminUser"
      MONGO_PASSWORD: "ZXgwB6pHmh0hVPLKAjZECSCBd9yucdTILxohUQ7QoGHrt7DwDp"
      MONGO_DB: "sidita"
      MONGO_DSN: "mongodb://adminUser:ZXgwB6pHmh0hVPLKAjZECSCBd9yucdTILxohUQ7QoGHrt7DwDp@103.175.221.64:27017/sidita?directConnection=true"
      MONGO_DB_NAME: "sidita"
      MONGO_COLLECTION_NAME: "logs"
```