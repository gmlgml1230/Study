# Raspberry Pi and Mopidy

## 개요

라즈베리파이에 Pi Musicbox를 설치하여 사용 중 출근, 점심 시간, 퇴근 시에 음악을 켜고 끄는 자동화 시스템에 대한 필요를 느꼈습니다. 하지만, Pi Musicbox는 설치는 편리하나 Extension 기능이 없기에 파이썬으로 작성된 확장 가능한 음악 서버인 Mopidy를 설치하게 되었습니다.

## 삽질 과정

- 도커 활용  
   Mopidy를 설치하기로 마음먹고 핫한 Docker를 활용해보고자 시도를 해봤습니다. 하지만, 도커를 활용하여 Mopidy를 설치한 사례가 많지 않았으며, OS를 Raspbian로 사용했기에 Docker 설치에도 많은 고생을 했습니다. 그렇기에 Docker를 내려놓고 직접 설치하여 사용하고자 결심했습니다.

## 개발 과정

1. Install Raspbian
   - Download URL
      > Raspbian iso 파일 다운로드 : https://www.raspberrypi.org/downloads/raspbian/  
      > SD Card Formatter 파일 다운로드 : https://www.sdcard.org/downloads/formatter_4/  
      > Win32 Disk Imager 파일 다운로드 : https://sourceforge.net/projects/win32diskimager/files/latest/download

   - Root Passwd 변경
      - 필수 사항은 아니지만 하는 것이 좋음
         ```
         sudo passwd root
         ```
1. Enable SSH on Raspbian
   - Raspberry Pi GUI의 Terminal을 사용하면 모니터와 키보드, 마우스를 따로 연결해주어야 하기 때문에 SSH 연결하여 사용하는 것이 편리함
   - SSH 허용(Raspbian)
      ```
      sudo raspi-config
      Interfacing Options -> SSH -> YES
      ```

   - IP 확인 방법(Raspbian)
      ```
      ifconfig
      ```

   - CMD 연결(VScode Terminal)
      - 참고 URL : https://extrememanual.net/12589
      ```
      pi [계정]@[아이피]
      ```

1. Install Mopidy
   ```
   1. Add the archive’s GPG key:
      wget -q -O - https://apt.mopidy.com/mopidy.gpg | sudo apt-key add -
   
   2. Add the APT repo to your package sources:
      sudo wget -q -O /etc/apt/sources.list.d/mopidy.list https://apt.mopidy.com/stretch.list
   
   3. Install Mopidy and all dependencies:
      sudo apt-get update
      sudo apt-get install mopidy

   4. Start Mopidy
      mopidy
   ```

1. Install Web Extension
   - Web Extension 종류 : https://docs.mopidy.com/en/latest/ext/web/
   - Mopidy-MusicBox-Webclient : https://github.com/pimusicbox/mopidy-musicbox-webclient
   - 설치 과정
      ```
      git clone https://github.com/pimusicbox/mopidy-musicbox-webclient
      cd mopidy-musicbox-webclient
      sudo python setup.py install
      ```
   - Mopidy Setting
      ```
      1. conf 파일 위치 찾기
         find / -name mopidy.conf

      2. Install vim
         apt-get install -y vim

      3. Mopidy 세팅 값 수정
         vim /root/.config/mopidy/mopidy.conf

         <추가>
         [musicbox_webclient]
         enabled = true
         musicbox = false
         websocket_host =
         websocket_port =
         on_track_click = PLAY_ALL

         <변경>
         [http]
         enabled = true
         hostname = [IP]
         port = 6680
         #static_dir =
         #zeroconf = Mopidy HTTP server on $hostname
         #allowed_origins =
         #csrf_protection = true
      ```

1. Mount USB Drive in Raspbian
   - USB에 음악파일을 넣어 사용하기 때문에 Raspberry Pi에서 USB Drive를 인식 할 수 있도록 해야한다.
   - 참고 영상 : https://www.youtube.com/watch?v=Q0W6ggl5yjY
   - Mount USB Drive
      ```
      sudo apt-get -install -y ntfs-3g
      
      # sda1 값 찾기 
      ls -l /dev/disk/by-uuid

      sudo mkdir /media/usb1
      id -g pi
      id -u pi
      sudo vim /etc/fstab

      # 내용 추가
      UUID=[sda1 값] /media/sub1 auto nofail,uid=1000,gid=1000,noatime 0 0

      sudo umount /dev/sda1
      sudo mount -a

      # 파일 확인
      ls /media/usb1
      ```

1. Mopidy Local Scan
   - Mopidy에서 Local File을 인식할 수 있어야 Web Browser에서 파일을 선택할 수 있음.
   - 참고 URL : https://docs.mopidy.com/en/latest/ext/local/#confval-local/media_dir
   - Mopidy Setting
      ```
      # Web Extension 추가 시 사용했던 파일과 다른 위치의 파일을 수정해야함
      vim /etc/mopidy/mopidy.conf

      # USB Drive Mount 시킨 위치 지정
      <변경>
      [local]
      media_dir = /media/usb1

      # Mopidy Local File Scan
      mopidy local scan
      ```
   - 위의 위치만 지정한다고 scan이 되지 않는 경우가 발생할 수 있다. Mopidy에서 해당 폴더를 읽을 수 있는 권한이 없기 때문일 수 있음. [권한 지정 방법](https://unix.stackexchange.com/questions/21251/execute-vs-read-bit-how-do-directory-permissions-in-linux-work)

1. Install mpc
   - MPD 제어 프로그램으로써 Mopidy는 MPD로 만들어져 mpc로 음악 재생 등 제어가 가능하다.
   - 참고 URL : https://jdm.kr/blog/2
   - Install mpc
      ```
      apt-get install -y mpc
      ```
   - 참고 URL : https://linux.die.net/man/1/mpc

1. Bash File 작성
   - Cron 작업 예약 시 실행 할 파일 작성
   - 참고 URL : https://www.ckdsn.com/how-to-a-spotify-alarm-clock-for-raspberry-pi/#mpc
   - cron_music (출근 시, 점심 시간 끝난 후 실행)
      ```
      #!/bin/bash

      mpc clear
      mpc load music
      mpc random on
      mpc play
      mpc volume 30
      ```
   - cron_lunch_music (점심 시간에 실행)
      ```
      #!/bin/bash

      mpc add file:///media/usb1/lunch/lunch2_gogibanchan.mp3
      mpc play 11
      mpc volume 40
      sleep 30
      mpc del 0
      mpc play 1
      sleep 1
      mpc stop
      ```

1. Install crontab
   - 아침, 점심, 저녁 시간에 자동 실행을 위해 필요한 도구
   - 참고 URL : https://www.raspberrypi.org/documentation/linux/usage/cron.md
   ```
   # Install crontab
   sudo apt-get install -y gnome-schedule

   # cron 작업 예약
   crontab -e

   # 출근 시간, 점심 시간 후 실행
   0 14,18 * * 0-4 sh /home/pi/cron_music
   # 점심 시간 알림
   45 16 * * 0-4 sh /home/pi/cron_lunch_music
   # 퇴근 시 정지
   0 23 * * 0-4 mpc stop
   ```

