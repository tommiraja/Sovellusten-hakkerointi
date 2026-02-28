# Hardware Hacking
## Tasks:
1. Decrypt firmware image
2. Analyse the image file
3. Extract rootfs from the image file
4. Search available applications
5. Analyse and try to open root passwords

## 1. Decrypting firmware image

- Alkuun kloonasin GitHub repon virtuaalikoneelleni `git clone https://github.com/robbins/tp-link-decrypt`
- Latasin AWS CLI:n `sudo apt install -y awscli`
- Latasin seuraavaksi TapoV3 Firmware binary `aws s3 cp s3://download.tplinkcloud.com/firmware/Tapo_C200v3_en_1.4.2_Build_250313_Rel.40499n_up_boot-signed_1747894968535.bin Tapo_C200v4_en_1.4.2.bin --no-sign-request`
- Latasin kameran dump-filen https://hhmoodle.haaga-helia.fi/mod/resource/view.php?id=3617382
- Luin `README.md` tiedostoa ja ajoin `./preinstall.sh` & `./extract_keys.sh`
- Tarvittavia työkaluja: `sudo apt install -y git build-essential` & `sudo apt install -y libssl-dev pkg-config`

 <img width="761" height="48" alt="image" src="https://github.com/user-attachments/assets/4b3abe3d-caa7-4bb2-9d3a-c3981fa2954e" />

 - Kokeilin komentoa `make`, mutta se ei toiminut sain vain seuraavaa erroria

<img width="930" height="178" alt="image" src="https://github.com/user-attachments/assets/7f906955-df81-491c-905b-4599a86cca7e" />

- Errori johtui siitä, että `extract_keys.sh` skripti ei luonut koskaan `RSA_0.h`
- Tässä vaiheessa kysyin tekoälyltä apua ja se esitti ratkaisuksi `venv`
- ...
- Ajoin `./extract_keys.sh example.jpg`

<img width="1010" height="361" alt="image" src="https://github.com/user-attachments/assets/0cc7d518-21b6-40d0-a355-5cbf2eea04a3" />

- Sitten ajoin komennot `make clean` & `make`

<img width="1280" height="216" alt="image" src="https://github.com/user-attachments/assets/b3c2c34b-7a00-4a81-b0d6-6022a109bf56" />

## 2. Analyzing the image file

- Tein bin tiedoston `make bin/tp-link-decrypt`
- Ajoin binwalkin `binwalk -eM firmware.decrypted.bin`

<img width="271" height="55" alt="image" src="https://github.com/user-attachments/assets/862888f0-afdc-4ce7-95bb-f7855d37d72a" />

- Uudet bin ja bin.dec tiedostot ilmestyivät, eli dekryptoitu versio tiedostosta onnistui
- Aloitetaan dekryptoidun tiedoston analysointi ajamalla `binwalk Tapo_C200V4_en_1.4.2.bin.dec` ja sen jälkeen puretaan sisätltö `binwalk -eM TapoC200v4_en_1.4.2.bin.dec`

<img width="949" height="206" alt="image" src="https://github.com/user-attachments/assets/62a4d017-f3d0-4785-9b05-abf2e4ace13f" />

<img width="409" height="55" alt="image" src="https://github.com/user-attachments/assets/6ad152ac-f5eb-4de5-ba3e-84fa1d705927" />

- Huomataan että seuraavat kolme tiedostoa syntyivät

## 3. Extracting Rootfs

- Siirrytään kansioon `cd _Tapo_C200v4_en_1.4.2.bin.dec.extracted/` ja katsotaan mitä se sisältää `ls -la`

<img width="810" height="144" alt="image" src="https://github.com/user-attachments/assets/b8171fb6-0a06-4b77-99f6-cf48aafb7266" />

- Squashfs-root näkyvissä, tutkitaan sitä!
- Etenin tiedostoon ja listasin kansion, katsoin `bin` tiedostoa ja huomataan, että executable ohjelmia, sekä `main` ohjelma näkyy, tätä ohjelmaa tulen todennäköisesti tutkimaan kun etsin salasanaa
- Tutkitaan nopeasti mitä filejä on kyseessä `file (TIEDOSTONIMI)` ja `strings`
    - Date -> buysbox & dmesg -> busybox ovat ilmeisesti rikkoutuneuita BusyBoxin symlinkkejä 
    - Main -> Pääohjelma, hyödyllinen Ghidrassa tms.
    - Gdbserver -> Debuggaus palvelin
    - Logcat -> Lokityökalu
    - 
<img width="1057" height="88" alt="image" src="https://github.com/user-attachments/assets/16c83e03-4688-4183-8ade-0ad080ea8484" />



 
