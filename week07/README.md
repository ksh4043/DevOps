# .env를 GitHub에 올리지 않는 이유
DB 정보(비밀번호), API Key 등 중요 정보를 노출시키지 않기 위해 올리지 않으며 대신 어떤 값이 필요한지 알려주는 .env.example 파일을 등록.


# docker compose ps 결과
NAME                IMAGE            COMMAND                  SERVICE   CREATED          STATUS          PORTS
guestbook-redis-1   redis:7-alpine   "docker-entrypoint.s…"   redis     56 minutes ago   Up 56 minutes   6379/tcp
guestbook-web-1     guestbook-web    "python app.py"          web       56 minutes ago   Up 56 minutes   0.0.0.0:8080->5000/tcp, [::]:8080->5000/tcp


# scale 후 각 웹 컨테이너 호스트명이 바뀐 화면 결과
~/DevOps/week06/guestbook$ docker compose up -d --scale web=3
[+] up 17/17
 ✔ Image nginx:alpine          Pulled                                                                         7.7s
 ✔ Network guestbook_default   Created                                                                        0.0s
 ✔ Container guestbook-redis-1 Healthy                                                                        6.0s
 ✔ Container guestbook-web-3   Healthy                                                                       11.9s
 ✔ Container guestbook-web-1   Healthy                                                                       11.9s
 ✔ Container guestbook-web-2   Healthy                                                                       11.9s
 ✔ Container guestbook-nginx-1 Started                                                                       11.9s


# down 후 up과 down -v 후 up 비교
## docker compose down
docker compose down
[+] down 6/6
 ✔ Container guestbook-nginx-1 Removed                                                                        0.4s
 ✔ Container guestbook-web-1   Removed                                                                       10.4s
 ✔ Container guestbook-web-2   Removed                                                                       10.5s
 ✔ Container guestbook-web-3   Removed                                                                       10.4s
 ✔ Container guestbook-redis-1 Removed                                                                        0.3s
 ✔ Network guestbook_default   Removed

## docker compose up -d
docker compose up -d
[+] up 3/3
 ✔ Container guestbook-redis-1 Healthy                                                                        5.6s
 ✔ Container guestbook-web-1   Healthy                                                                        5.7s
 ✔ Container guestbook-nginx-1 Started

## docker compose down -v
docker compose down -v
[+] down 5/5
 ✔ Container guestbook-nginx-1 Removed                                                                        0.4s
 ✔ Container guestbook-web-1   Removed                                                                       10.2s
 ✔ Container guestbook-redis-1 Removed                                                                        0.3s
 ✔ Volume guestbook_redis-data Removed                                                                        0.0s
 ✔ Network guestbook_default   Removed

## docker compose up -d
docker compose up -d
[+] up 5/5
 ✔ Network guestbook_default   Created                                                                        0.0s
 ✔ Volume guestbook_redis-data Created                                                                        0.0s
 ✔ Container guestbook-redis-1 Healthy                                                                        5.9s
 ✔ Container guestbook-web-1   Healthy                                                                       11.5s
 ✔ Container guestbook-nginx-1 Starte
