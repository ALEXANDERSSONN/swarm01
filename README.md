# REFERENCE

[github.com/docker/awesome-compose](https://github.com/docker/awesome-compose)


# SWARM-DEPLOY

[spcn04django.xops.ipv9.me](https://spcn04django.xops.ipv9.me/)

# WAKATIME
[WAKATIME](https://wakatime.com/@spcn04/projects/djmozndcbd)

# Setup-Linux
- Set เวลา โดยใช้คำสั่ง
```
sudo time datectl set-timezone Asia/Bangkok
```
 - คำสั่งที่ใช้ในการ Set ชื่อ Hostname
```
sudo hostnamectl set-hostname **ชื่อที่ต้องการจะตั้ง**
```
- คำสั่งที่ใช้ในการเปลี่ยน Machine-ID
```
rm /var/lib/dbus/machine-id
echo -n > /etc/machine-id
cat /etc/machine-id
ln -s /etc/machine-id /var/lib/dbus/machine-id
```

# INSTALL-DOCKER
- คำสั่งที่ใช้ลง Docker
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
- คำสั่งตรวจสอบการใช้งาน Docker
```
sudo docker run hello-world
```
# BUILD-IMAGE & TAG
- คำสั่งการ Build image
```
sudo docker compose "django/compose.yaml" up -d --build
```
- คำสั่งการ Tag
```
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```

# PUSH IMAGE TO DOCKER HUB 
- คำสั่งเข้าสู่ระบบ Docker ใน VSCODE
```
docker login
```
- คำสั่ง Push Image To Docker Hub
```
docker push TARGET_IMAGE[:TAG]
```

# CREATE STACK DEPLOY
- สร้างไฟล์ compose.yaml
```
version: '3.7'

services:
  db:
    image: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - db_data:/var/lib/postgresql/data

  web:
    image: TARGET_IMAGE[:TAG]
    volumes:
      - static_data:/usr/src/app/static
    ports:
      - "8088:8000"
    depends_on:
      - db
    deploy:
      replicas: 1
      restart_policy:
        condition: any
      update_config:
        delay: 5s
        parallelism: 1
        order: start-first

volumes:
  db_data:
  static_data:

networks:
  default:
    driver: overlay
    attachable: true
```
- นำ compose.yaml ไป Stack Deploy on local

# SWARM CLUSTER
- Revert Proxy compose.yaml
```
version: '3.7'

services:
  db:
    image: postgres
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    networks:
      - default
    volumes:
      - db_data:/var/lib/postgresql/data

  web:
    image: TARGET_IMAGE[:TAG]
    networks:
     - webproxy
     - default
    volumes:
      - static_data:/usr/src/app/static
    depends_on:
      - db
    deploy:
      replicas: 1
      labels:
        - traefik.docker.network=webproxy
        - traefik.enable=true
        - traefik.http.routers.${APPNAME}-https.entrypoints=websecure
        - traefik.http.routers.${APPNAME}-https.rule=Host("${APPNAME}$ URL ")
        - traefik.http.routers.${APPNAME}-https.tls.certresolver=default
        - traefik.http.services.${APPNAME}.loadbalancer.server.port=8000


      restart_policy:
        condition: any
      update_config:
        delay: 5s
        parallelism: 1
        order: start-first

volumes:
  db_data:
  static_data:

networks:
  default:
    driver: overlay
    attachable: true
  webproxy:
    external: true
```