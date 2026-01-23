# Break & Unbreak

x) Summary
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
