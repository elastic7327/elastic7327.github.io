---
comments: true
layout: post
title: Docker Swarm으로 손쉽게 마이크로 서비스를 만들어보자
excerpt_separator:  <!--more-->
---


# Docker  Swarm을 사용해서 Docker 오케스트레이션을 사용해보자

쿠버네틱스 또는 Mesoes를 사용하기 이전에 스타트업 초기단계(금전적으로 ㅎㄷㄷ일때)에서 사용하면 좋을 것 같기 때문에, aws 나노 인스턴스 3개를 띄운후에 진행 하려고합니다. 



물론 띄워놓고, terminate 하게되면 괜찮기 때문에, 상관 없다고 생각합니다.



이번에는 aws를 사용하려고합니다. 

https://aws.amazon.com/ 에 접속한 후에, 

프리티어 인스턴스 3개를 생성합니다. 



1개는 master로 사용할것이고, 

나머지 2개는 worker로 사용할 것입니다. 


![2018-07-17 5 04 06](https://user-images.githubusercontent.com/16227780/42824356-ac004ac0-8a1a-11e8-92f8-d6b224b3a977.png)


버지니아쪽이 인스턴스가 싸기때문에, 이런식으로 빠르게(?) 인스턴스 3개를 띄웁니다. 



첫번째 마스터 노드로 접근을합니다. 

 ssh로 붙어서 작업을 해봅니다. 

```bash
ssh -i "dockerswarmtest.pem" ubuntu@ec2-35-171-7-180.compute-1.amazonaws.com
```



일단 초기 상태이기 때문에, 도커를 적절하게 install 해줍니다.

```bash
# 이렇게 해주면 빠르게 도커만 설치가 됩니다. 
curl -fsSL get.docker.com -o get-docker.sh
sh get-docker.sh
```

그 이후. ifconfig를 통해서. Eth0 아이피 어드레스를 확인합니다.

```bash
eth0      Link encap:Ethernet  HWaddr 12:d7:8b:b6:ff:5e  
          inet addr:172.31.84.94  Bcast:172.31.95.255  Mask:255.255.240.0
          inet6 addr: fe80::10d7:8bff:feb6:ff5e/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:9001  Metric:1
          RX packets:39633 errors:0 dropped:0 overruns:0 frame:0
          TX packets:10500 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:56372623 (56.3 MB)  TX bytes:897360 (897.3 KB)


```

임시 실습에서 사용할 master 인스턴스의 ip는 172.31.84.94 이렇게 나왔습니다. 

여기서 우리는 2377 포트를 사용할것입니다.



```bash
docker swarm init --listen-addr=172.31.84.94:2377  

```

이렇게 입력을 하게되면 

```bash
docker swarm init --listen-addr=172.31.84.94:2377

Swarm initialized: current node (v5d1h6hb24ix401smme57rt5i) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-0fmgdi6jjakedgaxvy3psw2fepo1ecj7rdyx59urydbcwtsqdc-45n36k1t870odxw81uuewcmtx 172.31.84.94:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.


```



 적절하게 생성이 되었는지 확인을 하려면, docekr node ls 명령어로 확인을 합니다.

```bash


docker node ls       

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION

v5d1h6hb24ix401smme57rt5i *   ip-172-31-84-94     Ready               Active              Leader              18.05.0-ce
```



자! 이제 swarm 매니저를 성공적으로 등록했습니다! 



그리고 다음으로 다른 인스턴스(worker-1) 으로 접속을 합니다!.



~~이후 이런식으로 하게되면...~~

 `docker swarm join 172.31.84.94:2377`

~~타임아웃이 걸리거나, 동작하지 않습니다. 왜냐하면 포트가 열려있지 않은경우와 모르는 네트워크 이기 때문입니다.~~ 

~~또는 aws 보안그룹 정책 때문인데요, 보안그룹에 들어가서 인바운드 포트를 다 열어줍니다. 모든 트래픽을 받기로 합니다.~~ 

~~아이피는 위치무관으로 변경시켜줍니다.~~



여기서 첫번째 삽질 구간이 생겼는데,  처음! 인스턴스를 생성할때에, subnet 네트워크를 꼭 같은것으로 생성해야합니다. 

(아마도 이것 떄문에 많은 블로그에 포스팅된 글이, aws cli 사용하라고 되어있엇군요)

그리고 인스턴스를 1개 1개 생성하지말고, 인스턴스 숫자를 선택을하는 부분이 있는데 2개 또는 3개를 해서 생성을 합니다. 



그렇게 되면 

```bash
# 물론 이전에 ifconfig 로 확인을 해줍니다. 
docker swarm init --listen-addr 172.31.53.141:2377

Swarm initialized: current node (pxc8ei73d1grh92sex99nm4tk) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-35x43966ikt2vekeynpq9pj9ja89713q2xn172e47x5cexuadm-8m99yllb9bd8g82lfd8jg7yg9 172.31.53.141:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.


```



그리고 다른 aws 인스턴스에서 복붙을 해주면!

`docker swarm join --token SWMTKN-1-35x43966ikt2vekeynpq9pj9ja89713q2xn172e47x5cexuadm-8m99yllb9bd8g82lfd8jg7yg9 172.31.53.141:2377` 

`This node joined a swarm as a worker. `

이렇게 우리가 원하는 답을 내줍니다. 



이후에 master node 에 가서 docker node ls 명령어를 하게되면 

```
root@ip-172-31-53-141:/home/ubuntu# docker node ls

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION

pxc8ei73d1grh92sex99nm4tk *   ip-172-31-53-141    Ready               Active              Leader              18.05.0-ce

09h3khbz7anzw4miavo72dryq     ip-172-31-57-101    Ready               Active                                  18.05.0-ce
```

짠!! 우리가 원하는게 나왔습니다! 

자 그다음 우리는 nginx를 서비스하려고하는데요 



- Container:
  docker run -d nginx 를 통해서 하나의 이미지를 1개의 워커를 통해서 총10개를 띄워 보도록 하겠습니다(10개 re).  
- Service:
  - docker service create --replicas 10 nginx
  - 

docker service create --replicas 5 -p 80:80 --name web nginx 

이렇게 해줍니다. 그렇게되면, nginx가 5개가 떠있다는걸 확인할수 있습니다. 

```bash


docker service ps web

ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS

e94rr53zw4zb        web.1               nginx:latest        ip-172-31-57-101    Running             Running about a minute ago                       

mqwv6d32gfkb        web.2               nginx:latest        ip-172-31-53-141    Running             Running about a minute ago                       

7ba54v38rsgp        web.3               nginx:latest        ip-172-31-57-101    Running             Running about a minute ago                       

65j49ohgxl5e        web.4               nginx:latest        ip-172-31-53-141    Running             Running about a minute ago                       

sb3qd8zlcbz9        web.5               nginx:latest        ip-172-31-57-101    Running             Running about a minute ago          
```



스캐일을 올려봅니다.  래플리카를 5개에서 -> 8개로 올려봅니다. 

```

docker service scale web=8

web scaled to 8

overall progress: 8 out of 8 tasks 

1/8: running   [==================================================>] 

2/8: running   [==================================================>] 

3/8: running   [==================================================>] 

4/8: running   [==================================================>] 

5/8: running   [==================================================>] 

6/8: running   [==================================================>] 

7/8: running   [==================================================>] 

8/8: running   [==================================================>] 

verify: Service converged 
```



결과를 확인해봅니다. 

```


docker service ps web

ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS

e94rr53zw4zb        web.1               nginx:latest        ip-172-31-57-101    Running             Running 3 minutes ago                        

mqwv6d32gfkb        web.2               nginx:latest        ip-172-31-53-141    Running             Running 3 minutes ago                        

7ba54v38rsgp        web.3               nginx:latest        ip-172-31-57-101    Running             Running 3 minutes ago                        

65j49ohgxl5e        web.4               nginx:latest        ip-172-31-53-141    Running             Running 3 minutes ago                        

sb3qd8zlcbz9        web.5               nginx:latest        ip-172-31-57-101    Running             Running 3 minutes ago                        

whilldw8oi01        web.6               nginx:latest        ip-172-31-53-141    Running             Running 25 seconds ago                       

ujr8nlp97dla        web.7               nginx:latest        ip-172-31-53-141    Running             Running 25 seconds ago                       

mak9vo49qrqi        web.8               nginx:latest        ip-172-31-57-101    Running             Running 25 seconds ago        
```



마스터 노드에서 docker ps 를 통해서 하나 컨태이너 내부의 로그를 들여다본것이다. 

```
 docker logs -f  web.2.mqwv6d32gfkb5gstgx9ro0jfn

10.255.0.2 - - [17/Jul/2018:09:20:17 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36" "-"

10.255.0.2 - - [17/Jul/2018:09:20:18 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36" "-"


그리고 다른 워커 노드에서 docker ps를 통해서 하나 컨태이너 내부의 로그를 들여다 본것이다.
root@ip-172-31-57-101:/home/ubuntu# docker logs -f web.1.e94rr53zw4zbr278wddecet29
10.255.0.2 - - [17/Jul/2018:09:20:16 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36" "-"
10.255.0.2 - - [17/Jul/2018:09:20:18 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36" "-"
10.255.0.2 - - [17/Jul/2018:09:20:19 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_13_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36" "-"

저절로 로드밸런싱이 되고있다!
그리고 스케일링을 하고싶다면 swarm join을 해서 하나하나 늘려가면 될거같다.
한번 테스트해보자.
다시 인스턴스를 만들고 도커를 설치해보자.
```





인스턴스를 하나 더 만들었고,

![image](https://user-images.githubusercontent.com/16227780/42824391-bd540cbc-8a1a-11e8-99b6-a70b7ec7da19.png)


`docker swarm join --token SWMTKN-1-35x43966ikt2vekeynpq9pj9ja89713q2xn172e47x5cexuadm-8m99yllb9bd8g82lfd8jg7yg9 172.31.53.141:2377` 

저도 이렇게 올리면서 느낀거지만 security 부분에서 inbound 값을 수정해주지 않았는데도 바로 

해당 명령어를 하게되면서 붙게 되었습니다.

이렇게 확인을 해보면 잘 붙은것으로 확인할수 있다. 

```
root@ip-172-31-53-141:/home/ubuntu# docker node ls

ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION

pxc8ei73d1grh92sex99nm4tk *   ip-172-31-53-141    Ready               Active              Leader              18.05.0-ce

yab6tdqgtkgd8l045ws0wm3vz     ip-172-31-54-86     Ready               Active                                  18.05.0-ce

09h3khbz7anzw4miavo72dryq     ip-172-31-57-101    Ready               Active                                  18.05.0-ce
```



조금 심하게? 스케일업을 해보자 컨테이너를 20개를 띄워보자.

```


root@ip-172-31-53-141:/home/ubuntu# docker service scale web=20

web scaled to 20

overall progress: 20 out of 20 tasks 

1/20: running   [==================================================>] 

2/20: running   [==================================================>] 

3/20: running   [==================================================>] 

4/20: running   [==================================================>] 

5/20: running   [==================================================>] 

6/20: running   [==================================================>] 

7/20: running   [==================================================>] 

8/20: running   [==================================================>] 

9/20: running   [==================================================>] 

10/20: running   [==================================================>] 

11/20: running   [==================================================>] 

12/20: running   [==================================================>] 

13/20: running   [==================================================>] 

14/20: running   [==================================================>] 

15/20: running   [==================================================>] 

16/20: running   [==================================================>] 

17/20: running   [==================================================>] 

18/20: running   [==================================================>] 

19/20: running   [==================================================>] 

20/20: running   [==================================================>] 
```



방금전 명령어를 하기전에, 체크하고 넘어갈 부분이 있었다. 

swarm join 을 하는경우 그 직후에 해당 인스턴스(노드)에는 컨테이너가 자동(?)으로 분배 되지 않는 현상이 있었다. 

위의 스케일 명령어를 한 이후에는 적절하게 분배가 되었다. 



그리고 이런식으로 내가만든 서비스(컨테이너)를 한순에 쉽게 확인해볼수 있다.

```
docker service inspect web --pretty

ID:		a0uqf1o6qhmkqpzhua7xaexdf

Name:		web

Service Mode:	Replicated

 Replicas:	20

Placement:

UpdateConfig:

 Parallelism:	1

 On failure:	pause

 Monitoring Period: 5s

 Max failure ratio: 0

 Update order:      stop-first

RollbackConfig:

 Parallelism:	1

 On failure:	pause

 Monitoring Period: 5s

 Max failure ratio: 0

 Rollback order:    stop-first

ContainerSpec:

 Image:		nginx:latest@sha256:a461bb9fdeddb777053c0b50dbbe8a83d0f82f1c4b6de2b9c0445596fd7c0217

Resources:

Endpoint Mode:	vip

Ports:

 PublishedPort = 80

  Protocol = tcp

# 포트 80에 대해서 서비스를 하고 있는중이다.
  TargetPort = 80

  PublishMode = ingress 


```



도커 스웜을 통해서 저비용으로 고효율을 낼수 있는 예를 해보았다. 



마지막으로 잘 사용했으니.. 지워버리도록 합니다. 

![2018-07-17 7 07 39](https://user-images.githubusercontent.com/16227780/42824420-c7f654a4-8a1a-11e8-9fad-f8d849d81af9.png)
