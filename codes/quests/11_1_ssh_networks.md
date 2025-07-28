Network Shell Script 실습 문제
문제 환경 설정
실습을 위해 다음 파일들을 생성하세요:
### 1. 네트워크 로그 파일 생성
```shell
cat > network.log << 'EOF'
2024-01-15 10:30:25 192.168.1.100 CONNECT success
2024-01-15 10:30:30 192.168.1.101 CONNECT failed
2024-01-15 10:31:15 192.168.1.102 CONNECT success
2024-01-15 10:31:20 192.168.1.100 DISCONNECT success
2024-01-15 10:32:10 192.168.1.103 CONNECT success
2024-01-15 10:32:15 192.168.1.101 CONNECT success
2024-01-15 10:33:25 192.168.1.104 CONNECT failed
2024-01-15 10:33:30 192.168.1.102 DISCONNECT success
EOF
```
### 3. 접속 통계 파일 생성
```shell
cat > connections.txt << 'EOF'
192.168.1.100 5
192.168.1.101 12
192.168.1.102 8
192.168.1.103 3
192.168.1.104 15
192.168.1.105 7
EOF
```

# 문제 1: 네트워크 연결 상태 분석기
## 요구사항:
- network.log 파일을 분석하여 연결 성공/실패 통계를 출력하는 스크립트 작성
- 전체 연결 시도 수, 성공 수, 실패 수를 계산
- 성공률을 백분율로 표시 (소수점 제거)
- 출력 형태:
- === 네트워크 연결 분석 결과 ===
- 전체 연결 시도: X건
- 성공: Y건
- 실패: Z건
- 성공률: W%

## 제한사항:
- if문과 변수만 사용
- grep, wc, cut 명령어 활용
- 파일명은 스크립트 실행 시 첫 번째 인자로 받기
```shell
# remote to hwang's pc
[hwang@192.168.0.44 ~/Downloads]$ source network_analyzer.sh
# sh 파일 내용 EOF
#!/bin/bash

cat network.log

V_TRIED=$(($(cat network.log | wc -l)))
V_SUCCESS=$(($(cat network.log | grep "success" | wc -l)))
V_FAILED=$(($(cat network.log | grep "failed" | wc -l)))

V_PERCENT_SUCCESS=$(($V_SUCCESS * 100/$V_TRIED))

echo "All Tried : $V_TRIED"
echo "Success : $V_SUCCESS"
echo "Failed : $V_FAILED"
echo "Success Rate : $V_PERCENT_SUCCESS%"
# EOF



```
# 문제 2: IP 주소별 접속 빈도 상위 리스트
## 요구사항:
- network.log에서 IP 주소별 접속 횟수를 계산
- 접속 횟수 기준으로 내림차순 정렬하여 상위 3개만 출력
- 각 IP의 첫 접속 시간도 함께 표시
- 출력 형태:
- === 접속 빈도 TOP 3 ===
- 1위: 192.168.1.XXX (X회) - 첫 접속: 10:XX:XX
- 2위: 192.168.1.XXX (X회) - 첫 접속: 10:XX:XX  
- 3위: 192.168.1.XXX (X회) - 첫 접속: 10:XX:XX

## 제한사항:
- if문과 변수만 사용
- cut, sort, uniq, grep 명령어 활용
- head나 tail로 결과 제한

```shell
# sh 파일 내용 EOF (문제 1 sh 에 추가)
V_TOP3=$(cat $1 | cut -d" " -f3 | sort -t" " -k2 | uniq -c | sort -t " " -k1 -nr | head -n 3)
V_TOP1_IP=$(cat $1 | cut -d" " -f3 | sort -t" " -k2 | uniq -c | sort -t " " -k1 -nr | head -n 3 | cut -d" " -f8 | head -n 1)
V_TOP2_IP=$(cat $1 | cut -d" " -f3 | sort -t" " -k2 | uniq -c | sort -t " " -k1 -nr | head -n 3 | cut -d" " -f8 | head -n 2 | tail -n 1)
V_TOP3_IP=$(cat $1 | cut -d" " -f3 | sort -t" " -k2 | uniq -c | sort -t " " -k1 -nr | head -n 3 | cut -d" " -f8 | tail -n 1)
V_TOP1_TIME=$(cat $1 | cut -d" " -f2,3 | grep $V_TOP1_IP | head -n 1 | cut -d" " -f1)
V_TOP2_TIME=$(cat $1 | cut -d" " -f2,3 | grep $V_TOP2_IP | head -n 2 | tail -n 1 | cut -d" " -f1)
V_TOP3_TIME=$(cat $1 | cut -d" " -f2,3 | grep $V_TOP3_IP | tail -n 1 | cut -d" " -f1)
V_TOP1_COUNT=$(echo $V_TOP3 | cut -d" " -f1)
V_TOP2_COUNT=$(echo $V_TOP3 | cut -d" " -f3)
V_TOP3_COUNT=$(echo $V_TOP3 | cut -d" " -f5)
echo "TOP1. Trid IP : $V_TOP1_IP (count:$V_TOP1_COUNT) First connect time : $V_TOP1_TIME"
echo "TOP2. Trid IP : $V_TOP2_IP (count:$V_TOP2_COUNT) First connect time : $V_TOP2_TIME"
echo "TOP3. Trid IP : $V_TOP3_IP (count:$V_TOP3_COUNT) First connect time : $V_TOP3_TIME"
# EOF

[hwang@192.168.0.44 ~/Downloads]$ source network_analyzer.sh network.log
All Tried : 8
Success : 6
Failed : 2
Success Rate : 75%
TOP1. Trid IP : 192.168.1.102 (count:2) First connect time : 10:31:15
TOP2. Trid IP : 192.168.1.101 (count:2) First connect time : 10:32:15
TOP3. Trid IP : 192.168.1.100 (count:2) First connect time : 10:31:20

```

# 문제 3: 서버 상태 점검 스크립트
## 요구사항:
- servers.sh 실행해 각 서버에 대해 ping 테스트 실행
- 응답 있는 서버와 없는 서버를 구분하여 출력
- 응답 시간이 100ms 이상인 서버는 "느림" 표시
## 입력 형태:
	~$ servers.sh 123.92.0.12
## 출력 형태:
=== 서버 상태 점검 결과 ===
[정상] web01 (123.92.0.12) - 응답시간: XXms
OR
=== 서버 상태 점검 결과 ===
[오프라인] db01 (123.92.0.11) - 응답없음
...

## 제한사항:
if문과 변수만 사용
cut, ping 명령어 활용
ping은 1회만 실행 (ping -c 1)

```shell

```

문제 4: 네트워크 트래픽 임계값 모니터링
요구사항:
connections.txt에서 접속 수가 10 이상인 IP를 "높음", 5-9는 "보통", 4 이하는 "낮음"으로 분류
각 분류별로 개수 계산하여 출력
"높음" 분류의 IP들만 별도로 나열
출력 형태:
=== 트래픽 분석 결과 ===
높음(10회 이상): X개
보통(5-9회): Y개  
낮음(4회 이하): Z개

[주의 필요 IP 목록]
192.168.1.XXX (XX회)
192.168.1.XXX (XX회)

제한사항:
if문과 변수만 사용
cut, sort 명령어 활용
숫자 비교를 위한 조건문 사용

문제 5: 현재 시스템 네트워크 정보 수집기
요구사항:
현재 시스템의 IP 주소, 기본 게이트웨이, 활성 인터페이스 개수를 출력
인터넷 연결 상태 확인 (8.8.8.8로 ping 테스트)
모든 정보를 보기 좋게 정리하여 출력
출력 형태:
=== 시스템 네트워크 정보 ===
내부 IP: 192.168.1.XXX
기본 게이트웨이: 192.168.1.X
활성 인터페이스: X개
인터넷 연결: 정상/차단

제한사항:
if문과 변수만 사용
ip, hostname, ping, grep, wc 명령어 활용
각 정보를 변수에 저장 후 출력

실행 방법
각 문제의 스크립트를 작성한 후 다음과 같이 실행:
# 실행 권한 부여
chmod +x script_name.sh

# 스크립트 실행
./script_name.sh [인자]

주의사항
모든 스크립트는 #!/bin/bash로 시작
변수 선언 시 공백 없이 작성: var=value
if문 조건 확인 시 [ ] 사용
명령어 결과를 변수에 저장할 때 $(command) 사용
파일 존재 여부 확인: [ -f filename ]

