# davinci-docker
Davinci Docker Deployment


### Docker Build

```
docker build -t="edp963/davinci:v0.3.0-beta.4" .
```

### Docker

```
docker run -p 58081:8080 -e MYSQL_CONN="jdbc:mysql://docker.for.mac.host.internal:3306/davinci0.3?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true" \
-e DB_USER="root" -e DB_PWD="my-secret-pw" \
-e MAIL_HOST="smtp.163.com"  -e MAIL_PORT="465" -e MAIL_STMP_SSL="true" \
-e MAIL_USER="xxxxxx@163.com"  -e MAIL_PWD="xxxxxxx" \
-e MAIL_NICKNAME="davinci_sys" \
edp963/davinci:v0.3.0-beta.4
```

### Docker Compose

```
docker-compose up -d
```

```
docker-compose down -v
```


### ע������

docker-compose.yml ��������K=V�������пո�V��������˫���Ű���