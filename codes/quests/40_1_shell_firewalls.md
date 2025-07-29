🧪 문제 1: 특정 IP 차단 상태 확인 후 차단 설정
✅ 실행 예시
$ sudo ./problem1.sh
[INFO] 현재 rich rule 목록에 192.168.0.100 차단 룰이 존재하지 않습니다.
[INFO] 차단 룰을 추가합니다...
success

또는
$ sudo ./problem1.sh
[INFO] 192.168.0.100은 이미 차단되어 있습니다.
[SKIP] 추가 작업을 수행하지 않습니다.
```shell
# reject rule 존재 상태
[mincheolkim@192.168.0.36 ~/Downloads]$ sudo firewall-cmd --list-rich-rules
rule family="ipv4" source address="192.168.0.37" reject

[mincheolkim@192.168.0.36 ~/Downloads]$ source ./problem1.sh
[INFO] 192.168.0.37 is already rejected
[SKIP] No action

# reject rule 없는 상태
[mincheolkim@192.168.0.36 ~/Downloads]$ sudo firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="192.168.0.37" reject'
[mincheolkim@192.168.0.36 ~/Downloads]$ sudo firewall-cmd --list-rich-rules

[mincheolkim@192.168.0.36 ~/Downloads]$ 

[mincheolkim@192.168.0.36 ~/Downloads]$ source ./problem1.sh
[INFO] 192.168.0.37 is already rejected
[SKIP] No action

# sh 파일 내용 EOF
V_IP_RULE="192.168.0.37"

V_TEMP=$(sudo firewall-cmd --list-rich-rules | grep "$V_IP_RULE" | grep "reject" | wc -l)


if [ $V_TEMP -eq 0 ]; then
        echo "[INFO] There is no Rejection Rule from ip '$V_IP_RULE'"
        echo "[INFO] Add Rejection Rule to Firewall"
        sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="'$V_IP_RULE'" reject'
        sudo firewall-cmd --reload

else
        echo "[INFO] $V_IP_RULE is already rejected"
        echo "[SKIP] No action"
fi
# EOF

```

🔒 문제 2: 포트 8080이 열려 있다면 닫기
✅ 실행 예시
$ sudo ./problem2.sh
[INFO] 포트 8080/tcp 이 열려 있습니다. 제거합니다...
success

또는
$ sudo ./problem2.sh
[INFO] 포트 8080/tcp 이 열려 있지 않습니다. 아무 작업도 수행하지 않습니다.
```shell
# 열려있을 때 ( added 8080 port )
[mincheolkim@192.168.0.36 ~/Downloads]$ sudo firewall-cmd --permanent --add-port=8080/tcp
[sudo] password for mincheolkim: 
success
[mincheolkim@192.168.0.36 ~/Downloads]$ sudo firewall-cmd --reload
success
[mincheolkim@192.168.0.36 ~/Downloads]$ sudo firewall-cmd --list-ports
8080/tcp

[mincheolkim@192.168.0.36 ~/Downloads]$ source ./problem2.sh
[INFO] Port : 8080 is open. Now close.
success

# 닫혀있을 때 ( removed 8080 port )
[mincheolkim@192.168.0.36 ~/Downloads]$ sudo firewall-cmd --permanent --remove-port=8000/tcp
Warning: NOT_ENABLED: 8000:tcp
success
[mincheolkim@192.168.0.36 ~/Downloads]$ sudo firewall-cmd --reload
success
[mincheolkim@192.168.0.36 ~/Downloads]$ sudo firewall-cmd --list-ports

[mincheolkim@192.168.0.36 ~/Downloads]$ 

[mincheolkim@192.168.0.36 ~/Downloads]$ source ./problem2.sh
[INFO] Port : 8080 is not open. No action.

# sh 파일 내용 EOF
V_PORT_RULE="8080"

V_TEMP=$(sudo firewall-cmd --list-ports | grep "$V_PORT_RULE" | wc -l)

if [ $V_TEMP -eq 1 ]; then
        echo "[INFO] Port : $V_PORT_RULE is open. Now close."
        sudo firewall-cmd --permanent --remove-port=$V_PORT_RULE/tcp
        sudo firewall-cmd --reload
else
        echo "[INFO] Port : $V_PORT_RULE is not open. No action."
fi
# EOF


```





