# SSH와 원격 접속

## SSH 접속 (ssh user@host)
## SSH 키 인증 (ssh-keygen, authorized_keys)
## SSH config 설정 (~/.ssh/config)
## scp, rsync (파일 전송)
## SSH 터널링 (포트 포워딩)

---

# SSH (Secure Shell)
- 원격 호스트에 접속하기 위해 사용되는 보안 프로토콜
- 예전에는 telnet을 이용했는데 보안성이 떨어져서 지금은 거의 대부분 SSH를 이용

## SSH 설치

```shell
sudo apt update
sudo apt install openssh-server
sudo systemctl status ssh
```

## 방화벽에서 ssh 개방

```shell
sudo ufw allow ssh
```

## VirtualBox에서 포트 포워딩 설정
1. 호스트 IP: VirtualBox 설치된 컴퓨터 IP / 호스트 포트: 비는 포트
2. 게스트 IP: VirtualBox 설치된 IP / 게스트 포트: 22

## SSH 접속

```shell
# Mac
ssh 계정@127.0.0.1 -p 포트번호

# Windows: OpenSSH나 putty 설치해서 접속
ssh master@192.168.203.131 -p 10001
# 패스워드 입력
exit
```

---

# 서버 종료 및 재부팅

## 종료 : shutdown

```shell
shutdown -P +시간   # 시간(분) 후에 종료
shutdown -r 시간    # 시간에 종료
shutdown -c        # 예약한 shutdown 취소
shutdown -k +분    # 분 후에 종료 메시지 전송하지만 실제 종료 안 됨
```

## 재부팅

```shell
reboot
init 6
```
