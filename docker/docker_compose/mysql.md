## mysql 5.7

```yaml
version: '3'
services:
  mysql:
    image: mysql:5.7
    container_name: mysql57 
    environment:
      - MYSQL_ROOT_PASSWORD=root 
    ports:
        - "3306:3306"
```

## mysql 8

```yaml
version: '3.1'
services:
  mysql:
    image: mysql:8.0
    container_name: mysql8
    command: 
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
    # restart: always
    ports:
      - '3306:3306'
    environment:
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - './data/db:/var/lib/mysql'
      - ./data/conf:/etc/mysql/conf.d
      - ./data/logs:/logs
```