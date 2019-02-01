---
layout: post
title: "아파치 카프카 공식문서보고 따라해보기!!"
date: 2019-02-02 12:00:00
description:  # Add post description (optional)
img:  # Add image post (optional)
---

# Apache Kafka (아파치 카프카)
아파치 카프카(Apache Kafka)는 아파치 소프트웨어 재단이 스칼라로 개발한 오픈 소스 메시지 브로커 프로젝트이다. 이 프로젝트는 실시간 데이터 피드를 관리하기 위해 통일된, 높은 스루풋의 낮은 지연율를 지닌 플랫폼을 제공하는 것이 목표이다. 요컨대 분산 트랜잭션 로그로 구성된[^1], 상당히 확장 가능한 pub/sub 메시지 큐로 정의할 수 있으며, 스트리밍 데이터를 처리하기 위한 기업 인프라를 위한 고부가 가치 기능이다.
고성능 메시지 처리를 위한 Pub-Sub 모델의 메시지큐로써 2011년 링크드인이 Scala 언어로 개발 이후 오픈소스로 공개했다.


카프카의 경우 속도적 차이를 위해 페이징 캐쉬라는 것을 사용하고 있다.


# 용어
카프카에서 다루는 용어에 대해 간략히 소개하려고 한다.

topic (토픽) : 메시지가 분류되는 것, 일종의 채널로 볼 수 있다.
partiton (파티션) : 토픽을 나누는 것
offset(오프셋) : 메시지의 상대적인 위치를 나타내는 것이다.
broker (브로커) : 카프카의 서버를 칭한다. 브로커의 id는 integer형의 유니크한 값을 설정하여 한 노드에 여러 개 서버를 띄울 수 있다. 
zookeeper (주키퍼) : 주키퍼는 분산 메시지 큐 정보를 관리하는 역활을 한다.


# 카프카 따라해보기

## 1. 카프카 설치 하기 (install kafka )
카프카 바이너리 파일은 [여기](https://www.apache.org/dyn/closer.cgi?path=/kafka/2.1.0/kafka_2.11-2.1.0.tgz)에서 받을 수 있다.
```bash
> tar -xzf kafka_2.11-2.1.0.tgz
> cd kafka_2.11-2.1.0
```

## 2. 카프카 서버 기동하기

카프카는 z-node에 설정된 값을 설정으로 사용하기 때문에 주키퍼 서버부터 설정해준다.

```bash
   cd $ZOOKEEPER_HOME/bin
   ./zkServer.sh start
```

실행이 완료되면 아래와 같은 아웃라인이 나타난다.

```bash
   Starting zookeeper ... STARTED
```

아래와 같이 카프카를 실행한다

```
   cd $KAFKA_HOME
   bin/kafka-server-start.sh config/server.properties
```

## 3. 토픽 만들기

```
> bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic army
```

위와 같이 army라는 관심사(토픽)을 추가했다.

## 4. 카프카로 메시지 보내기

자 이제 관심사가 생겼으니 관심사에대한 데이터를 보내볼 시간이다.
아래와 같이 입력하면 카프카 토픽에 메시지를 보낼 수 있다.
필자는 관심사를 army 라고 했다

```bash
    bin/kafka-console-producer.sh --broker-list localhost:9092 --topic army
```

이러면 이제 입력을 할 수 있는 콘솔이 열리는데 필자는 아래와 같은 문자를 썼다

```bash
>이것은 수류탄이여!
>받아라 수류탄!!!
>전방 100m 적출현 사격 개시
```

필자는 유사군인이라 군생활은 하지 않았지만 그냥 넘어가도록 하자.

## 5. 카프카 토픽에서 메시지 꺼내보기

자 보냈다면 꺼내서 읽어봐야 순리에 맞지 않겠는가?
다음과 같은 명령어를 입력하여 카프카 토픽에서 아까 입력했던 메시지를 확인해보자.

```bash
   bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic army --from-beginning
```

자 위 명령어를 쳤다니 아까 넣었던 순서대로 나왔다.

```bash
   이것은 수류탄이여!
   받아라 수류탄!!!
   전방 100m 적출현 사격 개시
```

## 6. 카프카 브로커 다중 처리하기

여태까지 우리는 한 개의 브로커를 이용해서 클라이언트와 커뮤니케이션을 했었다.
자 그러면 이제 한 개 말고 여러개의 브로커로 클라이언트와 커뮤니케이션을 해보자!!

자 먼저 우리의 컨픽을 N개만큼 복사한다 필자는 3개로 진행할 것이다.
```bash
   cd config
   $ cp -r server.properties server1.properties
   $ cp -r server.properties server2.properties
```

이후 컨픽을 수정해야한다.

``` bash
vi server.properties

# --- server1.properties
# The id of the broker. This must be set to a unique integer for each broker.
broker.id=2
listeners=PLAINTEXT://:9093
log.dirs=/tmp/kafka-logs1

# --- server2.properties
# The id of the broker. This must be set to a unique integer for each broker.
broker.id=2
listeners=PLAINTEXT://:9094
log.dirs=/tmp/kafka-logs2
```

위의 컨픽 주석과 같이 브로커 id는 정수형 값을 가져야하며 동시에 유일해야한다라고 친절하게 적혀있다.
카프카 브로커마다 id를 다르게 해주자.

그후 위에 카프카 서버를 실행한 명령어와 같이 실행해준다
(물론 컨픽은 달라야한다!)

```bash
   bin/kafka-server-start.sh config/server1.properties &
   bin/kafka-server-start.sh config/server2.properties &
```

브로커를 띄운 후 3개의 브로커를 중계할 토픽을 생성한다.

```
   > bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic multiple
```

아래의 명령어는 토픽에 관련된 브로커의 상태들을 보여준다

자 우리는 방금 브로커를 실행시켰다. 하지만 지금 우리는 브로커가 제대로 기동됬는지에 알 수 있다

```bash
    > bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic multiple
    > Topic:multiple  PartitionCount:1        ReplicationFactor:3     Configs:
    >    Topic: multiple Partition: 0    Leader: 2       Replicas: 1,2,0 Isr: 2,0,1
```

위와 보면 우리가 새로만든 multiple 토픽에 3개의 브로커가 붙어있는걸 확인할 수 있다.

topic : 메시지가 이동할 타겟

partitionCount : 현재 토픽 내의 파티션 개수

isr : 현재 동기화되고있는 브로커의 집합이다.

Replicas : 이 파티션의 리더인지 혹은 현재 브로커가 동작하는지와 관련없이 현재 파티션의 로그를 남기는 노드 목록


자 이제 토픽을 열어서 메시지를 보내보자 

```bash
    > bin/kafka-console-producer.sh --broker-list localhost:9092 --topic multiple
    node1
    node2
    node
    node4
```

보냈으면 이제 메시지를 확인해보자

```bash
    > bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic multiple --from-beginning
    node1
    node2
    node
    node4
```
잘 나온다.

자 이제 브로커1을 죽여보자.

```bash
    seongju   4809  1.7  4.4 6774036 349940 pts/1  Sl   21:17   0:43 java -Xmx1G -Xms1G -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -Djava.awt.headless=true -Xloggc:/home/seongju/flatform/kafka_2.11-2.1.0/bin/../logs/kafkaServer-gc.log -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dkafka.logs.dir=/home/seongju/flatform/kafka_2.11-2.1.0/bin/../logs 

    kill -9 4809
```

윈도우에서는 아래와 같이 하도록 한다.
```cmd
    > wmic process where "caption = 'java.exe' and commandline like '%server-1.properties%'" get processid
    > ProcessId
    > 6016
    > taskkill /pid 6016 /f 
```

자 위에서 우리가 브로커를 죽였을때 토픽에 관한 상태를 다시 확인해보도록 하자
```bash
    bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic multiple
    Topic:multiple  PartitionCount:1        ReplicationFactor:3     Configs:
        Topic: multiple Partition: 0    Leader: 2       Replicas: 1,2,0 Isr: 2,0
```
우리가 죽인 브로커 1이 삭제되었다!

자 이제 표준입출력을 이용하여 카프카 메시지를 전송해보자

## 7. 카프카 파일로 메시지 보내기

```bash
    # 먼저 파일을 입력한다.

    echo "testing\nfilemessage" > message.dat
```

윈도우에서 메모장으로 만들거나 아래 명령어를 작성한다 

```cmd
    echo testing>> message.dat
    echo filemessage>> message.dat
```

그리고 설정 파일에 가서 다음 부분을 바꾼다

```bash
cd config
vi connect-file-source.properties

> topic = connect-test #AS IS
> topic = multiple #TO BE

> file = test.txt # AS IS
> file = message.dat # TO BE

vi connect-file-sink.properties

> topics=connect-test # AS IS
> topics=multiple # TO BE

> file=test.sink.txt # AS IS
> file=message.sink.dat #TO BE

```

```bash
    bin/connect-standalone.sh config/connect-standalone.properties config/connect-file-source.properties config/connect-file-sink.properties
```

파일 내용으로 보낸 메시지를 찾아보자

```bash
     bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic multiple --from-beginning

     {"schema":{"type":"string","optional":false},"payload":"testing\\nfilemessage"}
```

## 위의 글 힘드신 분을 위한 3줄 요약
 
1. 카프카는 고성능 메시지 큐(?)로 producer가 보내는놈이고 comsumer가 쓰는놈 토픽으로 메시지를 보낼 수 있음
2. 토픽 내에서 파티셔닝 가능
3. 파일 표준 입출력으로 메시지를 주고 받을 수 있음

스루풋 : 중앙 처리장치가 단위 시간에 처리할 수 있는 데이터 처리 능력을 일컫는 말


### 참고자료  
[카프카 공식페이지](https://kafka.apache.org/21/documentation/streams/quickstart)

[위키피디아:아파치 카프카](https://ko.wikipedia.org/wiki/%EC%95%84%ED%8C%8C%EC%B9%98_%EC%B9%B4%ED%94%84%EC%B9%B4)