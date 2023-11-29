## 상황
개발 배포 서버에서 카카오 소셜 로그인 수행시 다음과 같은 에러가 발생하였습니다.
<img width="901" alt="redis-error1" src="https://github.com/fine-ants/FineAnts-was/assets/33227831/f7b47ab4-01fb-4ff8-99d2-c70e9c4dec6e">
<img width="921" alt="redis-error2" src="https://github.com/fine-ants/FineAnts-was/assets/33227831/3f25d13d-450f-4a83-bd5d-4399465dc739">

## 원인
redis는 데이터 영속화를 위해서 BGSAVE 명령어를 이용하여 redis db의 스냅샷을 작성합니다. 그런데 BGSAVE 명령어가 실패하게 되면 Redis는 설정에 따라서 Write 명령어를 전부 거부하게 됩니다.

reids.c를 보면 processCommand에는 다음과 같은 코드가 존재합니다.
```c
if (server.stop_writes_on_bgsave_err &&
server.saveparamslen > 0
&& server.lastbgsave_status == REDIS_ERR &&
c->cmd->flags & REDIS_CMD_WRITE){
        flagTransaction(c);
        addReply(c, shared.bgsaveerr);
        return REDIS_OK;
 }
```
- 조건문에서 stop_writes_on_bgsave_err가 true인 경우 RDB 생성에 실패 했을 때 Write는 전부 거부되는 것을 확인할 수 있습니다. 

## 해결방법
redis를 캐시 용도로만 사용한다면 `config set stop-writes-on-bgsave-error no` 명령어를 통해서 RDB 스냅샷을 생성하지 않도록 합니다. 저희 인프라 같은 경우 docker-compose를 통해서 redis 컨테이너를 띄우기 때문에 docker-compose-dev.yml 파일에 볼륨을 이용해서 설정 파일을 마운트하여 RDB 스냅샷을 설정하지 않도록 합니다.

redis.conf
```
stop-writes-on-bgsave-error no
```

docker-compose-dev.yml
```
# ...
  redis:
    container_name: fineAnts_redis
    image: redis:latest
    volumes:
      - ./redis.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf
    ports:
      - "6379:6379"
    networks:
      - backend_net
 # ...
```

위 설정을 기반으로 docker-compose 실행시 redis-cli을 통해서 stop-writes-on-bgsave-error의 설정값이 "no"로 설정되었는지 확인할 수 있습니다.
```
# redis-cli
127.0.0.1:6379> config get stop-writes-on-bgsave-error
1) "stop-writes-on-bgsave-error"
2) "no"
```
- "1)" 은 설정의 이름을 의미하며 "2)"는 설정의 이름에 따른 현재값을 의미합니다.

### References
- https://charsyam.wordpress.com/2013/01/28/%EC%9E%85-%EA%B0%9C%EB%B0%9C-redis-%EC%84%9C%EB%B2%84%EA%B0%80-misconf-redis-is-configured-to-save-rdb-snapshots-%EC%97%90%EB%9F%AC%EB%A5%BC-%EB%82%B4%EB%A9%B0-%EB%8F%99%EC%9E%91%ED%95%98%EC%A7%80/
- https://stackoverflow.com/questions/19581059/misconf-redis-is-configured-to-save-rdb-snapshots


