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

<img width="779" height="417" alt="image" src="https://github.com/user-attachments/assets/8b382eaa-470f-4eec-96de-1917f9024875" />

<img width="896" height="308" alt="image" src="https://github.com/user-attachments/assets/974d78cb-c4de-47cb-bde0-8a880fd47846" />

<img width="1919" height="989" alt="image" src="https://github.com/user-attachments/assets/2d4835c2-2853-431a-a641-3625265f578c" />

<img width="926" height="568" alt="image" src="https://github.com/user-attachments/assets/d334cf1d-f1e2-4566-9f73-4333d71a36a8" />

<img width="935" height="593" alt="image" src="https://github.com/user-attachments/assets/320ce580-3cf7-4c89-9574-642cd81f1ecb" />

<img width="564" height="421" alt="image" src="https://github.com/user-attachments/assets/2dbf47c4-244c-468e-adb8-373e012caf9a" />


<img width="405" height="727" alt="image" src="https://github.com/user-attachments/assets/657f0803-76c4-46f2-b855-8d6754cb8963" />
- 0' OR 1=1 LIMIT 1 OFFSET 2 -- = SUPERADMIN%%rootALL-FLAG{Tero-e45f8764675e4463db969473b6d0fcdd}

## b) Lähdekoodin haavoittuvuuden korjaus:

<img width="959" height="932" alt="image" src="https://github.com/user-attachments/assets/bd5ba800-7617-40f6-8eda-13f30ba72ddf" />

<img width="576" height="171" alt="image" src="https://github.com/user-attachments/assets/3049c83c-43a9-4871-b5b7-6fccb0273f1a" />

<img width="959" height="571" alt="image" src="https://github.com/user-attachments/assets/86e1ea60-0fd0-4d44-87ff-055d97d7ab0a" />

<img width="388" height="464" alt="image" src="https://github.com/user-attachments/assets/d3b9a9b1-bc98-42f5-be6c-7cc05efb729b" />

## c) Piilotettujen verkkohakemistojen löytäminen ffuff:lla

<img width="948" height="721" alt="image" src="https://github.com/user-attachments/assets/5ac5d99d-57e3-4ba3-8ef7-aaa5b172f40b" />
















