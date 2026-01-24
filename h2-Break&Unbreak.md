# Break & Unbreak

## x) Summary
  - OWASP: OWASP Top 10: https://owasp.org/Top10/2021/A01_2021-Broken_Access_Control/index.html
    - Rikkinäinen pääsynhallinta luo merkittävän tietovuotoriskin, tällöin luvatonta tietoa voi paljastua, sitä voidaan muokata tai jopa tuhota
    - Yleisimpiä pääsynhallinnan haavoittuvuuksia: Least priviledge-oikeuden rikkoutuminen, API-pääsy ilman kattavaa tarkistusta, oikeuksien korotus, force browsing eli pakotettu sivustoselailu, metadatan manipulointi oikeuksien nostamiseksi. 
    - Miten pääsynhallintaan tunkeutuminen voidaan estää?: Oletuksen estäminen yksityisissä resursseissa, pääsynhallinnan on tärkeää pakottaa tietueen omistajuus, eikä olettaa käyttäjän oikeuksia luomiseen, lukemiseen ja päivittämiseen tai poistamiseen missä tahansa tietueessa. Pääsynhallinan epäonnistumiset on tärkeä lokittaa, jotta epämääräinen toiminta voidaan nähdä.

  - Karvinen 2023: https://terokarvinen.com/2023/fuzz-urls-find-hidden-directories/
    - Verkko palvelimilla on usein salattuja hakemistoja, joita ei ole linkitetty mistään paikasta, Fuff työkalun avulla voidaan lyötää näitä hakemistoja. Fuff on haavoittuvuuksien löytämiseen luotu ohjelma.
    - Fuzzing työkaluilla voi käyttää muihinkin kokonaisuuksiin esim. otsikkoihin tai POST-dataa
    - Fuzzer toimii automaattisesti, kokeillen tuhansia eri sanalistapolkuja

  - PortSwigger: https://portswigger.net/web-security/access-control
    - Pääsynhallinta päättää mitä oikeuksia käyttäjällä on, mitä käyttäjä saa tehdä tai päästä/nähdä resursseja virtuaaliympäristössä. Rikkinäinen pääsynhallinta on yleinen ja vaarallinen tietoturvallinen ongelma.
    - Pääsynhallinta sisältää kolme lajia:
        - Vertikaalinen: Tietyt roolit pääsevät tiettyihin resursseihin. Admin pystyy poistamaan käyttäjiä, normaali käyttäjä taas ei.
        - Horisontaalinen: Saman tasoiset käyttäjät pääsevät käsiksi vain omiin resursseihin, eivätkä näe muiden käyttäjien dataa/resursseja.
        - Kontekstiriippuvainen: Tietyt toiminnot käytössä tietynlaisessa tilassa tai järjestyksessä
    - Todennus (Authentication) vahvistaa käyttäjän olevan se, jota hän väittää olevansa
    - Istunnonhallinta (Session Management) tunnistaa käyttäjän tekemät HTTP-pyynnöt
    - Pääsynhallinta määrittää saako käyttäjä tehdä/suorittaa tietyn toiminnon virtuaaliympäristössä

  - Karvinen 2006: https://terokarvinen.com/2006/raportin-kirjoittaminen-4/
    - Viittaa lähteisiin raporttia tehdessä. Tämä osoittaa, että tekijä on perehtynyt aiheeseen.
    - Täsmällinen raportointi: Mitä komentoja käytin, onnistuiko, mitä tapahtui?
    - Raportin helppolukuisuus, huolellinen kirjoituskieli, väliotsikot, otsikot.

## a) Break into 010-staff-only.

Tehtävän tarkoituksena on löytää admin-salasana ja lipuke SQL-injektion avulla.

Aluksi latasin https://terokarvinen.com/hack-n-fix/ sivulta tehtävien zip-tiedoston, unzippasin sen ja pystytin serverin päälle. Minun tuli tehdä tämä hieman eritavalla käyttämällä porttiohjausta, sillä tein tehtävät Vagrantin avulla. 

<img width="779" height="417" alt="image" src="https://github.com/user-attachments/assets/8b382eaa-470f-4eec-96de-1917f9024875" />

<img width="896" height="308" alt="image" src="https://github.com/user-attachments/assets/974d78cb-c4de-47cb-bde0-8a880fd47846" />

<img width="1919" height="989" alt="image" src="https://github.com/user-attachments/assets/2d4835c2-2853-431a-a641-3625265f578c" />

Sitten oli aika selvittää admin pinniä, jotta löytäisin lipun. Kokeilin aluksi "0" & "123",  joka antoi salasanaksi "Somedude", sitten "321" joka antoi salasanaksi "foo".

Lähdin tutkimaan nettisivun selainkonsolia ja sieltä HTML-koodia. Huomasin kohdan `type=number` . Tarkoituksena oli päästä kirjoittamaan laatikkoon muitakin merkkejä, kuin numeroita vaihdoin sen siis `type=text` muotoon. 

Kokeilin kaikkia vinkeistä löytyviä kikkoja ja yhdistin niitä. mm:
- 0' OR 1=1 LIMIT 1 OFFSET 0 -- 
- 0' OR 1=1 LIMIT 1 OFFSET 1 -- 
- 0' OR 1=1 LIMIT 1 OFFSET 2 -- = Tämä SQL-injektio antoi oikean salasanan ja flagin.
 
<img width="926" height="568" alt="image" src="https://github.com/user-attachments/assets/d334cf1d-f1e2-4566-9f73-4333d71a36a8" />

<img width="935" height="593" alt="image" src="https://github.com/user-attachments/assets/320ce580-3cf7-4c89-9574-642cd81f1ecb" />

<img width="564" height="421" alt="image" src="https://github.com/user-attachments/assets/2dbf47c4-244c-468e-adb8-373e012caf9a" />


<img width="405" height="727" alt="image" src="https://github.com/user-attachments/assets/657f0803-76c4-46f2-b855-8d6754cb8963" />

- 0' OR 1=1 LIMIT 1 OFFSET 2 -- = SUPERADMIN%%rootALL-FLAG{Tero-e45f8764675e4463db969473b6d0fcdd}

## b) Lähdekoodin haavoittuvuuden korjaus:

Lähdin tutkimaan lähdekoodia katsomalla `cat staff-only.py` komennolla python tiedostoa. En oikeen aluksi saanut selvää, missä kohtaa haavoittuvuus ilmenee. Tutkin tehtäviin liittyviä vinkkejä ja löysin, että haavoittuvuus ilmenee kohdassa `SELECT password`.

<img width="959" height="932" alt="image" src="https://github.com/user-attachments/assets/bd5ba800-7617-40f6-8eda-13f30ba72ddf" />

- Vaihdoin SQL-koodia seuraavanlaiseen muotoon ja lisäsin `pin2 = ... `. 
- Aikaisemmin pin oli liitetty merkkijonona SQL:ään, joka mahdollisti SQL-kyselyn manipulointia ja lopulta admin salasanan paljastumisen.
- Korjattu SQL-pätkä muuttaa syötteen käsittelyä niin, että se on erillinen parametrinsa, tällöin se ei suoraan mene käsi kädessä osana SQL-komentoa vaan se käsitellään yksittäisenä tietona.

<img width="576" height="171" alt="image" src="https://github.com/user-attachments/assets/3049c83c-43a9-4871-b5b7-6fccb0273f1a" />

<img width="959" height="571" alt="image" src="https://github.com/user-attachments/assets/86e1ea60-0fd0-4d44-87ff-055d97d7ab0a" />

<img width="388" height="464" alt="image" src="https://github.com/user-attachments/assets/d3b9a9b1-bc98-42f5-be6c-7cc05efb729b" />

- Lopuksi kokeilin, että toimiiko tekemäni muutokset ja näytti toimivan! Sama SQL-injektio ei enään antanut admin salasanaa.

## c) Piilotettujen verkkohakemistojen löytäminen ffuff:lla

Alkuun latasin tarvittavat ohjelmat ja materiaalit tehtäviä varten ja kokeilin vain FFUFIN käyttöä tekemällä `dirfuzt-0` tehtävän. En selosta siitä sen tarkemmin, sillä `dirfuzt-1` on läksyjen kannalta olennainen tehtävä.

<img width="948" height="721" alt="image" src="https://github.com/user-attachments/assets/5ac5d99d-57e3-4ba3-8ef7-aaa5b172f40b" />

<img width="840" height="522" alt="image" src="https://github.com/user-attachments/assets/4825279c-46f3-49ce-bf42-a8e0596419b5" />

<img width="300" height="149" alt="image" src="https://github.com/user-attachments/assets/e5b4f3fb-f63b-4869-adfc-c3017f9f9203" />

<img width="872" height="313" alt="image" src="https://github.com/user-attachments/assets/7c6d8d85-aeee-4870-a1ca-f7bd34549ff1" />

Tässä tehtävässä etsin piilotettuja hakemistoja fuzzaamalla Ffufilla, joita ei näy selaimessa nettisivulla.

- `dirfuzt-1`-tehtävän tarkoituksena oli löytää admin sivun ja versionhallinta sivun URL-osoitteet, joista saa selville lipun.
- `dirfuzt-1`-tehtävän aluksi latasin teron sivulta zip-paketin tehtävään
- `$ sudo apt-get update` `$ sudo apt-get install ffuf`
- Latasin Seclististä wordlistin `wget https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/Web-Content/common.txt`
- Aloitin fuzzaasimen ajamalla `ffuff -w common.txt -u "http://127.0.0.2.8000/FUZZ" -fs 134`, jälkeen päin kokeilin samaa, mutta perään `-fs 154`
- Tämän jälkeen salaisia hakemistoja paljastui ja kokeilin kirjoittaa niitä URL:iin = /wp-admin onnistui, sekä .git. Nämä olivat tehtävän tavoitteita. Onnistui!
- Palvelimella voi olla salaisia hakemistoja, joita ei suoraan näy sivulla, tätä varten Ffufin käyttäminen toimii. Wordlisti ja Ffuf-fuzzing mahdollistaa automaattisen yleisten nimityksien kokeilun ja palauttaa CLI:hin vaihtoehtoja hakemistoista.
- Huomasin, kun `-fs` oli väärän kokoinen se palautti todella vastauksia, kun taas oikean kokoisena tulokset olivat pienempiä, mutta tarkempia.
- Lopulta varmistin hakemistot kirjoittamalla URL:iin Fuzzaamani hakemistot.

<img width="948" height="615" alt="image" src="https://github.com/user-attachments/assets/b775f30c-cd92-40ab-bfef-763cf0a32256" />

- /wp-admin = FLAG{tero-wpadmin-3364c855a2ac87341fc7bcbda955b580}
- /.git = FLAG{tero-git-3cc87212bcd411686a3b9e547d47fc51}

<img width="380" height="157" alt="image" src="https://github.com/user-attachments/assets/a68de531-2c9c-4fbd-bb89-a8ad70d75d7c" />

<img width="434" height="171" alt="image" src="https://github.com/user-attachments/assets/0ad26df0-3ce1-420a-b6b3-0e4fb103251f" />

## d) 020-Your-eyes-only

<img width="1142" height="549" alt="image" src="https://github.com/user-attachments/assets/1fb5436a-d88a-4cbe-bac9-44ecab35c2cb" />

<img width="535" height="274" alt="image" src="https://github.com/user-attachments/assets/38776641-e6eb-479d-b19d-d7b56e439ab6" />

<img width="453" height="355" alt="image" src="https://github.com/user-attachments/assets/dd876526-adb7-4eac-bbe4-ac628a570d38" />

## e) 020-Your-eyes-only haavoittuvuus

<img width="889" height="76" alt="image" src="https://github.com/user-attachments/assets/9afdd8bf-474f-4d74-b8f1-304334e5c57d" />

<img width="378" height="105" alt="image" src="https://github.com/user-attachments/assets/9edb990d-4f2d-4837-a725-729e241a9501" />

### Tehtäväyhteenveto & Lähteet



























