### ��������

```
git clone https://github.com/edp963/davinci-docker.git
cd /d davinci-docker
docker build -t="edp963/davinci:v0.3.0-beta.4" .
docker-compose up -d 
```

[http://localhost:58080](http://localhost:58080)


### Docker֧�ֻ��������б�

����|����|Ĭ��ֵ
-|-|-
HOST_DAVINCI|server.address��������|
MYSQL_CONN|datasource.url��jdbc mysql���Ӵ�|
DB_USER|datasource.username|
DB_PWD|datasource.password|
MAIL_HOST|mail.host
MAIL_PORT|mail.port
MAIL_USER|mail.username
MAIL_PWD|mail.password
MAIL_NICKNAME|mail.nickname
SMTP_TLS|mail.properties.smtp.starttls.enable|true
SMTP_TLS_REQUIRED|mail.properties.smtp.starttls.required|true
SMTP_AUTH|mail.properties.smtp.auth|true
MAIL_STMP_SSL|mail.properties.mail.smtp.ssl.enable|false

### ԭ�����

#### ����davinci docker����

**1. Dockfile����**
```
FROM java:8-jre

LABEL MAINTAINER="edp_support@groups.163.com"

# ��github�����طַ�������ѹ

RUN cd / \
	&& mkdir -p /opt/davinci\
	&& wget https://github.com/edp963/davinci/releases/download/v0.3.0-beta.4/davinci-assembly_3.0.1-0.3.0-SNAPSHOT-dist-beta.4.zip \
	&& unzip davinci-assembly_3.0.1-0.3.0-SNAPSHOT-dist-beta.4.zip -d /opt/davinci

# ��phantomjs���������

ADD phantomjs-2.1.1 /opt/phantomjs-2.1.1

# ���ݿ��ʼ���ű����ȴ����ݿ����������spring boot

ADD bin/start.sh /opt/davinci/bin/start.sh

# docker�����Ǿ�̬�ģ���������ļ��е�������Ҫ�û����������ݣ����12factor
# https://12factor.net/zh_cn/

ADD config/application.yml /opt/davinci/config/application.yml

# Ԥ��davinci�ر���������������
ENV DAVINCI3_HOME /opt/davinci
ENV PHANTOMJS_HOME /opt/phantomjs-2.1.1

WORKDIR /opt/davinci

# Ϊʲôʹ��CMD������ENTRYPOINT? ��ΪCMD������docker run��ʱ�����
# ��ʹ��compose��K8Sʱ�����п���Ҫ������ǰִ�������ű���������ֱ������
# start-server.sh
# �ڵ���docker run�Ҳ������κ�����ʱ����������Ĭ��ִ��

CMD ["./bin/start-server.sh"]

EXPOSE 8080
```

start.sh

```shell
#!/bin/bash

# ��sql�ű�����mysql8���ݴ����д��/initdbĿ¼
# /initdb Ŀ¼����mysql��������Ŀ¼
# mysql������������ʱִ�� /docker-entrypoint-initdb.d �е����нű�

cd /opt/davinci/bin/
mkdir /initdb
cat davinci.sql > /initdb/davinci.sql
sed -i '1i\SET GLOBAL log_bin_trust_function_creators = 1;' /initdb/davinci.sql


# ����docker compose������˳��������������Լ�
# ��� https://docs.docker.com/compose/startup-order/
# ���������Ҫ��curl̽��mysql�˿ڣ������������ֽڴ���0ʱ��Ϊ
# ���ݿ������ͨ������������ִ��davinci spring boot������
set -e

host="$1"
shift
cmd="$@"

until [ $(curl -I -m 10 -o /dev/null -s -w %{size_download} $host) -gt 0 ]; do
  >&2 echo "database is unavailable - sleeping"
  sleep 1
done

source $cmd
```

**2. ��������**

```
docker build -t="edp963/davinci:v0.3.0-beta.4" .
```

**3. docker compose**

```
version: '3.6'
services:
  davinci:
    environment:
      - MYSQL_CONN=jdbc:mysql://mysql:3306/davinci0.3?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true
      - DB_USER=root
      - DB_PWD=abc123123
      - MAIL_HOST=smtp.163.com
      - MAIL_PORT=465
      - MAIL_STMP_SSL=true
      - MAIL_USER=xxxxxx@163.com
      - MAIL_PWD=xxxxxxxx
      - MAIL_NICKNAME=davinci
    image: "edp963/davinci:v0.3.0-beta.4"
    ports:
      - 58080:8080
    # �ȴ�mysql������������spring boot������
    command: ["./bin/start.sh", "mysql:3306", "--", "start-server.sh"]
    restart: always
    volumes:
      - davinci_logs:/opt/davinci/logs
      - davinci_userfiles:/opt/davinci/userfiles
      - davinci_initdb:/initdb  #�����mysql�����ݳ�ʼ��
  mysql:
    image: mysql:8
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=abc123123
      - MYSQL_DATABASE=davinci0.3
    volumes:
      - mysql_data:/var/lib/mysql
      # ��ʼ���ű�Դ��davinic������initdbĿ¼
      - davinci_initdb:/docker-entrypoint-initdb.d:ro   

volumes:
  davinci_userfiles:
  davinci_logs:
  davinci_initdb:
  mysql_data:

    
```

*С��ʾ��docker-compose.yml������������K=V�в��ܳ��ֿո�VҲ������˫���Ű���*

**4. docker compose����**

```
docker-compose up -d 
```

**5.��docker����(ʹ���ⲿ���ݿ�)**

```
docker run -p 58081:8080 -e MYSQL_CONN="jdbc:mysql://yourmysqlserver:3306/davinci0.3?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&allowMultiQueries=true" \
-e DB_USER="root" -e DB_PWD="pwd" \
-e MAIL_HOST="smtp.163.com"  -e MAIL_PORT="465" -e MAIL_STMP_SSL="true" \
-e MAIL_USER="xxxxxx@163.com"  -e MAIL_PWD="xxxxxxx" \
-e MAIL_NICKNAME="davinci_sys" \
edp963/davinci:v0.3.0-beta.4
```

