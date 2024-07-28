# 42. DB Clustering



서버에 대한 가용성이 중요해진 만큼 밀접한 관계인 DB에 대한 고가용성도 필요해졌다.

서버에 대한 고가용성을 확보했더라도 DB에 대해 가용성이 확보되지 않는다면 DB는 SPOF가 될 것이다.

이를 위해 하나의 DB를 여러개의 서버로 구축할 필요가 생겼다. 이를 **DB Clustering**이라 한다.

DB를 여러개의 서버로 분산하면서 네트워크, DB서버의 자원에 대한 부하도 줄어드는 효과를 통해 성능적인 개선 또한 얻을 수 있다.





## DB Clustering 구현 방식

DB Clustering은 여러 개의 DB 서버가 하나의 DB 스토리지를 공유하며 작동하는 방식으로 구현된다.

여러 개의 DB 서버를 수평적으로 확장하여 Fail Over한 DB를 구축하는게 목표이다.



### Active - Active Clustering

![image](https://github.com/kauKoala/Backend-Study/assets/26706925/b8613b5c-2ed8-42c8-8383-7390df07e3de)


클러스터에서 두개 이상의 데이터베이스 서버를 항상 동작시키는 방식이다. 



장점으로는 한 서버가 다운되더라도 곧바로 다른 서버가 작동하기에 다운타임이 줄어든다.

또한 Load Balancing을 도입하여 한 DB서버에 집중되는 부하를 분산할 수도 있을것이다.



단점으로는 DB 스토리지 하나를 공유하므로 DB 스토리지에 대한 여러 DB 서버의 IO로 병목이 발생할 수 있으며 

항상 두대 이상의 서버를 동작시켜야 하기에 자원적인 비용이 증가한다.



### Active - Standby Clustering

![image](https://github.com/kauKoala/Backend-Study/assets/26706925/e513c991-bb75-4902-9104-8186a74cbc8c)


데이터베이스 서버 하나를 동작시키고 나머지 서버는 대기상태로 두는 방식이다. 동작중인 서버가 다운될 경우 대기중인 서버 중 하나를 동작시킨다.



장점으로는 Active - Active에 비해 적은 비용드는 점이다.



단점으로는 Active - Active방식보다 다운타임이 길다.



### 그냥 공유디스크 안쓰면 안되나?

DB Clustering은 기본적으로 여러개의 데이터베이스 서버가 하나의 데이터 집합을 가져야 한다.

만약 중복되는 DB 데이터를 분산하여 저장하고 있다면 이는 **Replication**이다. 

공유디스크를 사용하지 않고 중복되지 않는 DB 데이터를 분산하여 저장하는 방식은 **샤딩**이라 한다. 



### MySQL

Mysql 에서는 NDB Cluster, InnoDB Cluster등을 통해 DB Clustering을 지원한다.



### PostgreSQL

PostgreSQL에서는 Pgpool-II을 통해 DB Clustering을 지원한다.



### Oracle DB

Oracle DB는 RAC(Real Application Clusters)를 사용하여 DB Clustering을 지원한다. 





## 출처

https://medium.com/@su_bak/%EB%8D%B0%EC%9D%B4%ED%84%B0%EB%B2%A0%EC%9D%B4%EC%8A%A4%EC%97%90%EC%84%9C-%ED%81%B4%EB%9F%AC%EC%8A%A4%ED%84%B0-cluster-%EB%9E%80-9f3aa611f22a

https://c-yhhhhh.tistory.com/147

