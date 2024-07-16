# Docker
<hr/>
* By Docker & Docker-compose, we will install InfluxDB and Grafana

### Contents

[1. Install Docker](#install-docker)

[2. Install Docker-Compose](#install-docker-compose)

[3. Setting Docker-Compose](#setting-docker-compose)

<hr/>

## Install Docker

* Reference
	* <https://docs.docker.com/engine/install/ubuntu/>
	
* Install
	* `sudo apt install ca-certificates curl`
	* `sudo install -m 0755 -d /etc/apt/keyrings`
	* `sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc`
	* `sudo chmod a+r /etc/apt/keyrings/docker.asc`
	* `echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] 
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" 
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`
	* `sudo apt update`
	* `sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

* Service start
	* `sudo systemctl start docker`
	* `sudo systemctl enable docker`

* Add user to Docker group
	* `sudo usermod -aG docker cupadmin`

## Install Docker-Compose

* Reference 
	* <https://docs.docker.com/compose/>
	* <https://docs.docker.com/compose/reference/>

* Download the current stable release of Docker-Compose (Check latest version)
* V2.28.1 (2024. Jul)
	* <https://github.com/docker/compose/releases>
	* `sudo curl -L "https://github.com/docker/compose/releases/download/v2.81.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

* Apply executable permissions to the binary
	* `sudo chmod +x /usr/local/bin/docker-compose`

## Setting Docker-compose
* https://doc.owncloud.com/server/next/admin_manual/installation/docker/#docker-compose
* `cd /data`
* `mkdir docker-compose-owncloud; cd docker-compose-owncloud`
* `touch docker-compose.yml`

* docker-compose.yml

```
services:
  owncloud:
    image: owncloud/server:latest
    container_name: owncloud_server
    restart: always
    ports:
      - "8080:8080"
    depends_on:
      - mariadb
      - redis
    environment:
      - OWNCLOUD_DOMAIN=localhost:8080
      - OWNCLOUD_TRUSTED_DOMAINS=localhost
      - OWNCLOUD_DB_TYPE=mysql
      - OWNCLOUD_DB_NAME=owncloud
      - OWNCLOUD_DB_USERNAME=owncloud
      - OWNCLOUD_DB_PASSWORD=owncloud
      - OWNCLOUD_DB_HOST=mariadb
      - OWNCLOUD_ADMIN_USERNAME=cupadmin
      - OWNCLOUD_ADMIN_PASSWORD=cupibs2018!
      - OWNCLOUD_MYSQL_UTF8MB4=true
      - OWNCLOUD_REDIS_ENABLED=true
      - OWNCLOUD_REDIS_HOST=redis
      - PHP_UPLOAD_MAX_FILESIZE=16G
      - PHP_POST_MAX_SIZE=16G
    healthcheck:
      test: ["CMD", "/usr/bin/healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 5
    volumes:
      - ./oc-data:/mnt/data

  mariadb:
    image: mariadb:latest
    container_name: owncloud_mariadb
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=owncloud
      - MYSQL_USER=owncloud
      - MYSQL_PASSWORD=owncloud
      - MYSQL_DATABASE=owncloud
      - MARIADB_AUTO_UPGRADE=1
    command: ["--max-allowed-packet=512M", "--innodb-log-file-size=64M"]
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-u", "root", "--password=owncloud"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - ./mysql-data:/var/lib/mysql

  redis:
    image: redis:latest
    container_name: owncloud_redis
    restart: always
    command: ["--databases", "1"]
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 5
    volumes:
      - ./redis-data:/data

```

* Check point
	* `- OWNCLOUD_ADMIN_USERNAME=cupadmin`
	* `- OWNCLOUD_ADMIN_PASSWORD=cupibs2018!`
	* `- PHP_UPLOAD_MAX_FILESIZE=16G`
	* `- PHP_POST_MAX_SIZE=16G`

* Change Permission

	* `sudo chmod 666 /var/run/docker.sock`
 
* Start docker-compose service

	* `docker-compose up -d`

* Modify PHP config (Large file size problem)
	* `sudo emacs -nw /data/docker-compose-owncloud/oc-data/config/config.php`
		* `  'max_filesize' => '16G',`
	* `sudo docker restart <container_id>`