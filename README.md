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