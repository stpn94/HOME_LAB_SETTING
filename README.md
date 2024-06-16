# HOME_LAB_SETTING
HOME_LAB_SETTING

## 0.서버 셋팅
```
1. 서버 구매
- 당근에서 아무거나 구매 (DELL PowerEdge_T410 12만원에 구매)

2. 하이퍼 바이저 선택
- proxmox 선택
- vmware이 더 친숙하긴함.... vmware ESXi는 유료서비스 및 지원하지 않는 서비스가 있다고 들었음.
- windowServer2008은 cd_key가 있는데 너무 옛날꺼임.
- proxmox 정보가 많지 않지만 나름 사용자들 사이에서 추천하는거 같음. 사용하기 편해보였음.

3. OS 선택
- Ubuntu 선택
- 항상 centOS를 사용 했는 이제 지원안하는거 같음.

type-1
hyper v - proxmox
os - ubunto

4. rufus-4.5 다운로드
- 간편하게 부팅 가능한 USB 드라이브 만들기

5. proxmox-ve_8.2-1.iso 다운로드
- rufus 돌리기

__ ※HDD서버가  인식이 안됨.
BIOS 업데이트 - 소용없음
여러가지 펌웨어 업데이트 - 소용없음
디스크 접촉불량 - 아님

알고보니 피지컬 드라이브에서 가상드라이브 만들어 줘야함
그리고 RAID0 사용__






## 1. Proxmox VE 설치 후 초기 업데이트
```
# Proxmox 설치 후, 최신 보안 업데이트 및 소프트웨어 업데이트를 수행해야 합니다.
# 언제: 설치 직후
# 어디서: 터미널에서
# 무엇을: 시스템 패키지 업데이트
# 어떻게: apt 명령어를 사용하여 업데이트 수행
# 왜: 최신 보안 패치 적용 및 소프트웨어 안정성 향상

apt update
apt full-upgrade -y

# 업데이트 후 시스템 재부팅이 필요할 수 있습니다.
# 시스템 재부팅 명령어:
reboot
```



## 1. No-Subscription 저장소 추가
```
# Proxmox는 기본적으로 Enterprise 저장소를 사용합니다. 이는 구독 없이 접근할 수 없으므로, No-Subscription 저장소를 추가해야 합니다.
# 언제: Proxmox 설치 직후
# 어디서: /etc/apt/sources.list.d/pve-no-subscription.list 파일 생성
# 무엇을: No-Subscription 저장소 추가
# 어떻게: 파일에 저장소 주소 추가
# 왜: 구독 없이 업데이트 및 패키지를 설치하기 위해

# 파일 생성 및 저장소 주소 추가
echo "deb http://download.proxmox.com/debian/pve buster pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list

# 패키지 리스트를 업데이트합니다.
apt update

# 저장소 목록 확인
cat /etc/apt/sources.list.d/pve-no-subscription.list
```
![image](https://github.com/stpn94/HOME_LAB_SETTING/assets/79563672/d9ae6740-f7e3-48f1-8ed2-6b6c3c2f3ee0)

## 2. Enterprise 구독 비활성화
```
# 기본적으로 설정된 Enterprise 저장소를 비활성화합니다.
# 언제: Proxmox 설치 직후
# 어디서: /etc/apt/sources.list.d/pve-enterprise.list 파일
# 무엇을: Enterprise 저장소 주석 처리
# 어떻게: 파일 내용을 주석 처리
# 왜: 구독 팝업을 제거하고, 구독 없이 업데이트를 받기 위해

# 파일 내용을 주석 처리
sed -i.bak "s/^deb/#deb/" /etc/apt/sources.list.d/pve-enterprise.list

# 변경 사항 확인
cat /etc/apt/sources.list.d/pve-enterprise.list

# 변경된 저장소 리스트를 적용하기 위해 패키지 리스트를 업데이트합니다.
apt update
```
명령어와 확인 방법(저장소 추가 후 업데이트 확인)
```
# 패키지 리스트를 업데이트하여 저장소가 제대로 추가되었는지 확인합니다.
apt update

# 설치 가능한 패키지 리스트를 확인합니다.
apt list --upgradable
```
## 2. Subscription 팝업 제거
```
# Proxmox는 기본적으로 비상업용 무료 사용자를 위한 구독 팝업을 표시합니다.
# 언제: Proxmox 설치 후 초기 설정 시
# 어디서: /etc/apt/sources.list.d/pve-enterprise.list 파일
# 무엇을: 구독 팝업 제거
# 어떻게: 파일의 내용을 주석 처리
# 왜: 불필요한 팝업을 제거하여 사용성을 높이기 위해

sed -i.bak "s/^deb/#deb/" /etc/apt/sources.list.d/pve-enterprise.list

# 저장소 목록을 추가하여 업데이트를 진행합니다.
echo "deb http://download.proxmox.com/debian/pve buster pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list

# 변경 사항을 적용하기 위해 패키지 리스트를 업데이트합니다.
apt update
```
## 3. 네트워크 설정
```
# 네트워크 설정을 통해 Proxmox 서버의 IP 주소를 고정할 수 있습니다.
# 언제: 초기 설정 시
# 어디서: /etc/network/interfaces 파일
# 무엇을: 고정 IP 설정
# 어떻게: 인터페이스 설정 파일을 편집
# 왜: 안정적인 네트워크 연결을 위해

nano /etc/network/interfaces

# 예시 설정:
# 기존 설정 주석 처리 후 추가
# auto eth0
# iface eth0 inet static
#     address 192.168.1.100
#     netmask 255.255.255.0
#     gateway 192.168.1.1
#     dns-nameservers 8.8.8.8 8.8.4.4

# 설정 적용을 위해 네트워크 인터페이스를 재시작합니다.
systemctl restart networking

# 네트워크 설정 확인:
ip addr show eth0
```

## 4. 시간대 설정
```
# Proxmox 서버의 시간대를 정확하게 설정합니다.
# 언제: 초기 설정 시
# 어디서: 터미널에서
# 무엇을: 시간대 설정
# 어떻게: timedatectl 명령어를 사용
# 왜: 정확한 시간 동기화 및 로그 관리를 위해

timedatectl set-timezone Asia/Seoul

# 설정된 시간대 확인:
timedatectl
```

## 5. NTP 서버 설정
```
# 시간 동기화를 위해 NTP(Network Time Protocol) 서버를 설정합니다.
# 언제: 초기 설정 시
# 어디서: /etc/systemd/timesyncd.conf 파일
# 무엇을: NTP 서버 설정
# 어떻게: NTP 서버 주소 추가
# 왜: 정확한 시간 동기화 유지

nano /etc/systemd/timesyncd.conf

# 예시 설정:
# [Time]
# NTP=0.kr.pool.ntp.org 1.asia.pool.ntp.org

# 설정을 적용하고 서비스를 재시작합니다.
systemctl restart systemd-timesyncd

# NTP 동기화 상태 확인:
timedatectl status
```
## 6. SMTP 설정
```
# Proxmox에서 이메일 알림을 받기 위해 SMTP 서버 설정을 합니다.
# 언제: 초기 설정 시
# 어디서: /etc/ssmtp/ssmtp.conf 파일
# 무엇을: SMTP 서버 설정
# 어떻게: SMTP 서버 정보 추가
# 왜: 시스템 알림 및 경고 메일을 수신하기 위해

nano /etc/ssmtp/ssmtp.conf

# 예시 설정:
# root=your-email@example.com
# mailhub=smtp.example.com:587
# AuthUser=your-email@example.com
# AuthPass=your-email-password
# UseTLS=YES
# UseSTARTTLS=YES

# 설정을 테스트하기 위해 메일을 전송합니다.
echo "Test email from Proxmox" | mail -s "Proxmox Test Email" your-email@example.com

# 메일 로그 확인:
tail -f /var/log/mail.log
```

## 7. ZFS 파일 시스템 설정 (옵션)
```
# ZFS 파일 시스템을 사용하여 데이터의 신뢰성과 성능을 높일 수 있습니다.
# 언제: 초기 설정 시
# 어디서: 터미널에서
# 무엇을: ZFS 풀 생성 및 마운트
# 어떻게: zpool 명령어 사용
# 왜: 고성능 및 고신뢰성 스토리지 관리

# ZFS 풀 생성 (예: 2개의 디스크 사용)
zpool create mypool mirror /dev/sda /dev/sdb

# ZFS 풀 상태 확인:
zpool status mypool

# ZFS 파일 시스템 마운트:
mkdir /mnt/mypool
zfs set mountpoint=/mnt/mypool mypool

# 마운트 상태 확인:
df -h | grep mypool
```
