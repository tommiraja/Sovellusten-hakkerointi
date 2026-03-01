# Hardware Hacking
## Tehtävät:
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

- Errori johtui siitä, että `extract_keys.sh` skripti ei luonut koskaan `RSA_0.h`. Virtuaalikoneeltani puuttui tiettyjä purkutyökaluja, jonka seurauksena RSA-headerien generointi ei toiminut.
- Tässä vaiheessa kysyin tekoälyltä apua ja se esitti ratkaisuksi `venv` eli virtuaaliympäristön luominen, johon asennetaan tarvittavat työkalut
- Loin projektille oma virtuaaliympäristön `python3 -m venv .venv`, aktivoin sen -> `source.venv/bin/activate` ja sitten asensin ympäristöön tarvittavat purkutyökalut `ubi-reader, jefferson`. Tämän jälkeen projektin kääntäminen onnistui komennolla `make`.
 
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

## 3 & 4. Extracting Rootfs

- Siirrytään kansioon `cd _Tapo_C200v4_en_1.4.2.bin.dec.extracted/` ja katsotaan mitä se sisältää `ls -la`

<img width="810" height="144" alt="image" src="https://github.com/user-attachments/assets/b8171fb6-0a06-4b77-99f6-cf48aafb7266" />

- Squashfs-root näkyvissä, tutkitaan sitä!
- Etenin tiedostoon ja listasin kansion, katsoin `bin` tiedostoa ja huomataan, että executable ohjelmia, sekä `main` ohjelma näkyy, tätä ohjelmaa tulen todennäköisesti tutkimaan kun etsin salasanaa
- Tutkitaan nopeasti mitä filejä on kyseessä `file (TIEDOSTONIMI)` ja `strings`, en kuitenkaan saanut selkoa stringsien kautta ja kaikki tiedostost olivat elf-32 executable, nämäkään eivät kertoneet kaikkea. Hyödynsin tekoälyä tässä vaiheessa ja kysyin "Mikä merkitys seuraavilla ohjelmilla voisi olla?
    - Date -> buysbox & dmesg -> busybox ovat ilmeisesti rikkoutuneuita BusyBoxin symlinkkejä 
    - Main -> Pääohjelma, hyödyllinen Ghidrassa salasanan kräkkäämisessä tms.
    - Gdbserver -> Debuggaus palvelin
    - Logcat -> Lokityökalu
    - Getcpuinfo -> Mahdollisesti laitevalmistajan apuohjelma, joka lukee laitteen CPU:ta
    - Impdbg -> Debuggaustiedosto, joka sisältää logien dumpit ja testikomennot, taikka
      kamerasensorin/encoderin testaustiedosto
 
<img width="1057" height="88" alt="image" src="https://github.com/user-attachments/assets/16c83e03-4688-4183-8ade-0ad080ea8484" />

- Tutkin vielä tulostettavia stringsejä `main` ohjelmasta `strings main | grep -i "password"`

<img width="1281" height="717" alt="image" src="https://github.com/user-attachments/assets/7d1f5033-39f9-489e-95c0-ed00cc36e30c" />

- Kiinnitin huomiota kohtaan `"Hash(password) = %s` -> Tämä viittaa siihen, että salasanaa käsitellään hashina eli salasanan tallennus tapahtuu mahdollisesti hashattuna (salattuna)
- Huomataan myös kohdat `Encoding password: %s` ja `update_password` ehkä root-salasana generoidaan automaattisesti ja päivitetään?
- Katsoin vielä main ohjelmaa `strings main | grep -i "root"`,

<img width="1250" height="561" alt="image" src="https://github.com/user-attachments/assets/918fb5a1-1720-47d8-b03c-18c7f0767f49" />

<img width="470" height="381" alt="image" src="https://github.com/user-attachments/assets/56f9116e-9a8f-42ed-b289-29fc3a94a8ea" />

<img width="905" height="549" alt="image" src="https://github.com/user-attachments/assets/88397b6c-b5d1-430d-bff7-5f86154bc4c8" />

- Kuvien perusteella voidaan päätellä, että salasana käsitellään kryptografisesti ja ei ole selkokielisenä binäärissä. Kysyin tekoälyltä mikä tarkoitus `verifyThirdAccount`, `changeThirdAccount`, `getThirdAccount`, `setThirdAccount`, riveillä voisi olla. Sain vastaukseksi, että ne viittaavat kolmannen osapuolen käyttätiliin, mutta tästä ei kuitenkaan selvinnyt mitään kriittistä salasanan kräkkäämisen kannalta
- Salasana on todennäköisesti piilotettu, sillä mitään oikeannäköisiä arvoja ei saatu selville stringsien avulla.


## 5 & 6. Available applications and password hunt.
- Käytettävät ohjelmat käytiin äsken läpi, lähdetään etsimään salasanaa
- Tutkitaan vielä muita `squashfs-root` hakemistoja 
- En meinannut vieläkään löytää mitään tärkeää
- Kokeilin greppailla main ohjelmaa `strings main | grep -i 'passwd'`

 <img width="792" height="602" alt="image" src="https://github.com/user-attachments/assets/cd6100ab-74ac-428f-9d70-bbfa4a261dbf" />

 - Outputti antoi polun `/tmp/temp_passwd`, tämä voisi viitata väliaikaisen salasanan käyttöönotto tai palautus toiminnolta, sekä toinen kohta joka merkitsi huomioni oli `factory_passwd` eli tehdas-salasanaa, joka voisi viitata root-salasanaan.

- Päätin tässä vaiheessa ottaa Ghidran käyttöön, jos se paljastaisi meille jotain mielenkiintoista
- Avasin Ghidran `./ghidraRun` -> New project -> Import file -> ...main

<img width="849" height="648" alt="image" src="https://github.com/user-attachments/assets/68abfbb6-dab0-45ee-ae8f-d13a901e8c86" />

- Suoritin auto analyysin
- Menin entry pointtiin ja huomasin decompiler-ikkunassa funktion `FUN_0040bdb0`, tämä aloittaa main ohjelman logiikan, joten klikkasin sitä ja nimesin sen "TAPO_MAIN" funktioksi

<img width="503" height="431" alt="image" src="https://github.com/user-attachments/assets/e11a0fa9-5eca-4028-92dc-ad45ece8e855" />

- Ghidra käsittely jää vielä keskeneräiseksi tältäosin, sillä en kerkeä selvittämään tehtävää ennen deadlineä.
- Selvitän huomenna 1.03.2026 salasanaa tarkemmin!

## 1.03.2026

- Tutkin main ohjelmaa Ghidrassa aika pitkään koitin vain scrollata läpi analyysiä ja tutkia kaikkia mahdollisia funktioita
- Löysin funktion `FUN_0042d918` ja sen sisältä muita mielenkiintoisia funktiota. Tämä funktio käsittelee mm. käyttäjätunnusta, salasanaa ja ciphertekstiä, joten uskon tämän olevan merkittävä löytö

<img width="745" height="584" alt="image" src="https://github.com/user-attachments/assets/de1d9196-a652-420f-a1f8-de55846a9c19" />

- Tämän jälkeen löysin vielä `gen_root_passwd` funktion
- Decompile ikkunassa oli monia mielenkiintoisia polkuja "/user_management/hub_auth", "/user_management/root", en kuitenkaan saanut näistä apua tai inspiraatiota etsimään salasanaa syvemmin.

<img width="1178" height="570" alt="image" src="https://github.com/user-attachments/assets/1d8c1a28-141f-428e-882a-78bfbcd8dabb" />

<img width="587" height="308" alt="image" src="https://github.com/user-attachments/assets/bf045d23-963a-4727-9c10-325c72a426b0" />

## Lähteet:
- https://quentinkaiser.be/security/2025/07/25/rooting-tapo-c200/
- HHmoodle sovellusten hakkerointi ja haavoittuvuudet kurssi
- Tp-link-decrypt GitRepo https://github.com/robbins/tp-link-decrypt
- OpenAI, ChatGPT 5.2 tekoälymallia hyödynnetty  firmware imagen dekryptoinnin error-kohdissa, Squashfs-root ohjelmien merkityksen selvittämisessä, sekä 

