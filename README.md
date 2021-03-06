# :books:Bi-LSTM기반 실시간 Link 예측 모듈(KOREN_LinkPrediction)

KOREN망의 각 Link별 현재 상태를 바탕으로 향후 10분, 30분, 60분 후의 Link 상태를 예측하는 Bi-LSTM기반 인공지능 모듈


## :heavy_check_mark:Developer Environment

  - Language: [:crocodile:Python 3.7](https://www.python.org/)
  - IDE Tool: [:zap:Pycharm](https://www.jetbrains.com/pycharm/)
  - Package Manager: [:snake:Anaconda](https://www.anaconda.com/)
  - AI Library: Tensorflow2, KERAS
  - Collector: Beautifulsoup4
  - Storage: MongoDB, ElasticSearch
  - Visualization: Kibana
  
## :heavy_check_mark:Requirement
프로그램 실행 전 반드시 MongoDB를 설치해야 한다. [Mongodb 다운로드](https://www.mongodb.com/try)  
MongoDB를 설치하고 mongod 서버를 실행하기만 하면 된다. 기본적인 환경은 Default(Port: 27017)를 사용한다.

다음은 Linux(Ubuntu)환경에서 MongoDB를 설치하고 실행하는 방법이다.
```
$ sudo apt-get intsll -y mongodb-org
$ sudo service mongod start
$ ps -ef | grep mongodb
```

## :book:Model Process
### 1.데이터 수집
Beautifulsoup4 라이브러리를 활용하여 다음 6개의 KOREN망 Link정보를 수집한다.
  - timestamp
  - current_tx_packetpersecond
  - current_tx_bitpersecond
  - accumulated_tx_bytes
  - accumulated_tx_packets
  
KOREN망은 10개의 노드와 22개의 Link로 구성되며 다음과 같다.
![캡처](https://user-images.githubusercontent.com/28920880/105176608-cc76db80-5b68-11eb-919d-ea62dd01c4c7.PNG)

### 2.데이터 전처리
수집된 Link정보를 전처리한다.
  - timestamp => (hour x 60) + Minute = time_index
  - accumulated_tx_bytes => 현재 - 10분전 = 10분동안 송신 된 bytes(tx_bytes)
  - current_tx_bitpersecond => 평균 = 10분 동안 송신된 bps 평균(tx_bitpersecond)
  - accumulated_tx_packetpersecond => 현재 - 10분전 = 10분동안 송신 된 pps 평균(tx_packetpersecond)
  - tx_link_utilization => 1 - tx_link_utilization = Link 가용량(link_availability)
  
전처리 후 생성되는 데이터
  - time_index
  - tx_bytes
  - tx_bitpersecond
  - tx_packetpersecond
  - link_availability
  
### 3.데이터 저장
전처리 된 데이터를 MongoDB에 저장한다.

### 4.Link 예측
  - 인공지능 모델: Bi-LSTM기반 다음 10분 후 Link 상태를 예측하는 모델
  - 입력 데이터: 현재를 기준으로 이전 10분단위 30개 데이터(총 300분)
  - 출력 데이터: 다음 10분 후의 Link 상태

다음 10분을 예측할 수 있는 인공지능 모델을 재귀적으로 사용하여 다음 그림과 같이 Multi-step Prediction 한다.
![캡처](https://user-images.githubusercontent.com/28920880/105175807-c3d1d580-5b67-11eb-9f4b-ff276b35bc43.PNG)


### 5.예측 결과(예시)
예측 결과에 사용된 모델의 세부사항은 다음과 같다.
  - 링크 인터페이스: P2-Daejeon-prs1e11-Daejeon-Gwangju
  - 데이터셋: 7469개(10분 단위)
  - 데이터 수집 기간: 2020.12.12.T05:39 ~ 2020.12.22.T22:59
  - Train/Test 분류: Train(5982), Test(1487)
  - Epoch: 100
  - Batch-size: 100
  
대전-광주 Link의 10분 후 예측 결과는 다음 그림과 같다.
![캡처](https://user-images.githubusercontent.com/28920880/105176285-67bb8100-5b68-11eb-944b-15090be7be13.PNG)


### 6.시각화
예측 결과를 ElasticSearch에 저장한다. ElasticSearch에서 예측결과를 불러와 Kibana(웹페이지)에 시각화한다.

다음 그림은 대전-광주 Link의 Link 예측결과를 Kibana에 시각화한 결과다.
![캡처](https://user-images.githubusercontent.com/28920880/105176199-465a9500-5b68-11eb-9d76-14bb2b031439.PNG)

