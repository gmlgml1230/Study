## Elasticsearch 구축

### 설치 과정

### 1. 구축 환경
- Server : naver cloud platform - ubuntu 16.04

### 2. 설치 과정
1. Java 8 설치
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 0x219BD9C9
apt_source='deb http://repos.azulsystems.com/debian stable main'
apt_list='/etc/apt/sources.list.d/zulu.list'
echo "$apt_source" | sudo tee "$apt_list" > /dev/null
sudo apt-get update
sudo apt-get install zulu-8
```

- 설치 되었는지 확인 방법
```
java -version
```

2. elasticsearch 설치
```
# PGP Key를 이용하여 패키지를 암호화해 놓으므로 key를 받은 후 설치해야한다.
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
# APT를 이용해 설치하려면 필요한 패키지
sudo apt-get install apt-transport-https
# repository 정보를 해당 파일에 기록
echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list
# elasticsearch 설치
sudo apt-get update && sudo apt-get install elasticsearch
```

3. 환경 설정
```
vi /etc/elasticsearch/elasticsearch.yml
# 해당 파일에서 network.host의 주석처리 제거 후 localhost 대신 0.0.0.0 으로 변경
network.host: 0.0.0.0
```

4. elasticsearch 실행 방법
- 시스템 시작 시 자동 실행 시키는 방법
```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```

- 수동으로 켜고 끄는 방법
```
sudo systemctl start elasticsearch.service
sudo systemctl stop elasticsearch.service
```

5. 외부 접속 전 정상적으로 실행되었는지 확인 하는 방법
```
# default port : 9200
curl localhost:9200
```

6. Kibana 설치
```
apt-get install kibana
vi /etc/kibana/kibana.yml
# 해당 파일에서 server.host: "0.0.0.0" 으로 변경해야 외부 접속 가능

systemctl daemon-reload
# systemctl enable kibana
# enable 실행 시 부팅 시 자동 실행
systemctl start kibana
```

### 3. 접속 방법
1. Elasticsearch - localhost:9200
2. Kibana - localhost:5601

### 4. 데이터 적재 및 Kibana 사용 방법 참고 사이트
- 엘라스틱 설치 및 kibana 설치 참고
http://blog.naver.com/PostView.nhn?blogId=indy9052&logNo=220943826345&categoryNo=66&parentCategoryNo=0&viewDate=&currentPage=1&postListTopCurrentPage=1&from=postView
> X-Pack 플러그인은 보안용으로 설치하지 않아도됨
> 또한, 많은 환경설정변경을 제안하지만 해당 부분에 대해선 단일 노드 구성이 아닌 여러 노드를 사용할 시에 적용(단일 노드로 구성 시 위의 기록한 환경 변수만 수정하면 됨)

- Kibana Index Patterns 설정 방법
https://www.elastic.co/guide/en/kibana/current/tutorial-define-index.html
> 데이터 적재 시 사용한 index 명 + "\*" 으로 검색하면 된다(ex. test*)

- Data Mapping
http://blog.leekyoungil.com/?p=434
<br>
- ID 인덱스 지정 해주는 방법(R 패키지: elastic 사용)
http://www.iainmwallace.com/2018/01/21/elasticsearch-and-r/
> ID 인덱스를 지정하지 않고 데이터 적재 시 임의의 값으로 할당된다.
> 해당 값이 임의로 정의되어도 문제는 없으나, 후에 특정 값을 Update할 때 찾는데 혼란을 야기할 수 있다.
> 따라서, 날짜 및 특정 고유값을 묶어 Key값(ID)로 할당

- elasticsearch Package 사이트
https://cran.r-project.org/web/packages/elastic/elastic.pdf
<br>
- Kibana 시각화 방법
https://www.elastic.co/guide/kr/kibana/current/tutorial-visualizing.html
