---
layout: post
title: Tyrada o nowych pomysłach Allegro.pl
modified:
categories: ecommerce/allegro
description: Komentarz do zmian z minionego roku na portalu aukcyjnym Allegro.pl - REST, nowe miniatury, wielowariantowość
tags: [allegro, article, rant]
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2018-02-14T08:55:43+01:00
---

## Allegro, why? (zmiany od 2017)

﻿Niedawne zmiany na naszym ukochanym portalu aukcyjnym - Allegro.pl, wybrzmiały szerokim echem wśród prowadzących działalność gospodarczą opierającą się głównie na sprzedaży internetowej (biznesy z naciskiem na ~~jedyny~~ najpopularniejszy portal aukcyjny w Polsce). Niestety, nie był to radosny śpiew skowronka, lecz przypominający raczej jęk, odgłos ostatniego tchnienia, wielu zarzynanych przedsiębiorców. Gdyby zliczyć wszystkie zbrodnie tego portalu przeciwko sprzedawcom, prawdopodobnie staneliby przed sądem w Hadze.

<!-- more -->

Ostatni rok był szczególnie dotkliwy dla osób próbujących oferować tam swoje produkty, a to za sprawą kierunku jaki obrał tenże, niegdyś aukcyjny, portal. Obecnie wchodząc na Allegro.pl, coraz częściej odnoszę wrażenie, że omyłkowo znalazłem się na portalu innego giganta - Amazon. Nie chcę jedynie krytykować, bo jako konsument cieszy mnie wprowadzony standard zdjęć i ich miniatur, nowy system wielowariantowości, widoczne i wymagane dla każdej oferty karty z informacjami o gwarancji, zwrotach; nawet system komentarzy, który z początku wydawał się pomysłem... takim sobie, realnie pomaga w odnajdowaniu godnych zaufania kontrahentów. 

A co z połówką, której imię "przedsiębiorca"? Oprócz zniesienia opłat za wystawienie aukcji (konta firmowe) w niektórych kategoriach, ma LEKKO pod górę z nowymi pomysłami forsowanymi przez monopolistę. Okazało się bowiem, że każdy sklep musi nagle wydać niemałe pieniądze na nowe zdjęcia na białym tle i to dla WSZYSTKICH swoich ofert (nawet nie sięgam po kalkulator, koszta są ogromne), a to dopiero początek. Największym wyzwaniem okazuje się "nowy opis" na aukcjach. Pozwala on jedynie na używanie w nim zdjęć zaimportowanych przy tworzeniu aukcji - korzystając z formularza na stronie jeszcze nie jest tak źle, edytor jest czytelny i łatwo się go obsługuje, ale co z tymi, których asortyment to więcej niż 100 ofert i do tej pory musieli posiłkować się automatyzacją? 

### REST API

Allegro z dobroci serca oddało do tego celu nowe REST API... ale aukcje nadal wystawiamy przez stare toporne WebAPI (WSDL), czyli tak na prawdę tylko rzucono kolejną kłodę pod nogi. Owe REST API nie udostępnia zasobów do wystawiania aukcji. Co więcej, zamiast wprowadzić nowy zasób do starego WEBAPI, zmuszeni jesteśmy do skorzystania z serwisu REST, tylko po to, aby uzyskać "identyfikatory warunków oferty na danym koncie" - czyli te ładne karty z gwarancjami, warunkami zwrotów itd. 

> O co taki szum, se po prostu walne requesta do resta i mam te identyfikatory. - pomyślałem

No tak, ale REST API Allegro, mimo iż uważam za dobry krok, to w obecnej formie jest jedynie zbędnym utrudnieniem. Zmuszeni jesteśmy jednocześnie obsługiwać WSDL jak i REST API. 

Można nawet skorzystać z REST do autentykacji w starym WebApi, proces jest *banalnie* prosty:

> W celu ułatwienia (sic!) procesu logowania, udostępniliśmy w WebAPI metodę doLoginWithAccessToken, która na wejściu wymaga token oAuth i zwraca identyfikator sesji. Najpierw powinieneś zalogować się do REST API (przekazując login i hasło), a następnie wykorzystując access_token zalogować się do WebAPI (bez konieczności podawania loginu i hasła użytkownika). 

#### Chcesz tokena? Usiądź, bo troche to zajmie:
1. Rejestracja aplikacji: [Platforma do rejestracji aplikacji allegro.pl](https://credentials.allegroapi.io/)
2. Uzykane dane dostępowe wykorzystaj jako parametry żądania HTTP do zasobu uwierzytelniającego (otwierając przy tym graficzną przeglądarkę)
3. W przeglądarce widzisz znajomy formularz, logujesz się do swojego konta na allegro.pl
4. Wyrażasz zgodę na dostęp danej aplikacji do twojego konta
5. Pamiętasz punkt 1? Podawałeś tam *redirect_uri* - był to adres na który ma zostać zwrócony *kod autoryzujący* ("hola, hola to co ja musze jakiś serwer stawiać i odbierać żądania?" krótko: tak)
6. Gottagofast - masz teraz 10 sekund (jak na filmach z bombą) aby przesłać dalej uzyskany kod wykonując żądanie POST na odpowiedni adres.
7. PROFIT! Nareszcie upragniony JSON z naszym TOKENEM (ważnym przez 12 godzin - ale spokojnie, na 365 dni możesz zapomnieć o procedurze którą przed chwilą przeszedłeś, od teraz gdy obecny token wygaśnie, możesz użyć uzyskany w odpowiedzi - refresh_token, dzięki któremu uzyskasz nowy, świeżutki i żyjący również 12 godzin - access_token).

#### Czy jest źle? 

No nie do końca, bo jest to [OAuth 2.0](https://oauth.net/2/) i ~~niestety~~ będziemy musieli się przyzwyczaić, bo kolejne serwisy w ten sam sposób uwierzytelniają klienta. 

Kiedy uporamy się już z procesem uwierzytelniania i uzyskiwania tokenów, możemy zacząc bez przeszkód korzystać z łatwodostępnych i bogatych zasobów REST API... NIESTETY NIE! Obecnie REST Allegro wystawia mniej niż 10 zasobów, z których najistotniejszym jest *after-sales-service-conditions* (bez którego nie wystawimy aukcji z nowym opisem).

Nie krytykuję OAuth 2 ani flow tego typu autoryzacji, raczej jest to krytyka tego, że nadal bez względu na REST API, użyć musimy WSDL - do którego można zalogować się podając po prostu login i hasło, a dostęp do REST API na obecną chwile daje wątpliwe korzyści. Nie możemy więc w pełni zautomatyzować całego procesu, potrzebujemy ingerencji użytkownika na etapie logowania jak i serwer który odbierze token (*redirect_url*).

Więc o co tyle zachodu skoro i tak będziemy korzystać z zasobów starego WebAPI? Jak pisałem wcześniej, jest to etap którego nie możemy pominąć, jeżeli chcemy wystawić aukcję z nowym opisem. 

**Nota bene wykorzystanie tokena do autentykacji w WSDL jest bezpieczniejsze niż używanie starej metody, czyli loginu i hasła do konta allegro, więc uznaję to jako ~~główny~~ jedyny argument "ZA" używaniem REST w obecnej formie.**

### Wielowariantowość

A co z [wielowariantowością](https://pomoc.allegro.pl/info/jak_przejsc_na_wielowariantowosc)? Czy można w jakiś sposób masowo i automatycznie łączyć aukcje ze sobą? Przez API ? Odpowiedź brzmi:

<figure class="center">
	<img src='{{ site.url }}/images/allegro/rant1.jpg' alt="">
	<figcaption>źródło: FAQ Allegro.pl</figcaption>
</figure>

Allegro nie widzi problemu, uważa że regulamin który pośrednio wymusza, aby warianty przedmiotów były na osobnych aukcjach (miniatura może przedstawiać tylko jeden, dostępny "od ręki" wariant) nie jest przeszkodą. Zgodziłbym się, gdyby nie doświadczenia z towarami, które po prostu muszą być wystawione w wielu wariantach (buty, dywany, odzież). 

Za czasów opisów wspierających HTML, przedsiębiorcy radzili sobie z tym na różne sposoby, najpopularniejszym było wstawianie jako miniatury (zdjęcia poglądowego wyświetlanego na liscie aukcji) zbiorczego obrazu, przedstawiającego wszystkie warianty (kolor, wzór, wielkości), a w opisie, umieszczanie kilku galerii z podpisami kodu czy nazwy konkretnego wariantu i prośba o przekazanie wyboru w mailu / komentarzu do formularza pozakupowego. I to działało. 

Obecna sytuacja komplikuje wszystko, bardzo komplikuje. Cóż zrobić kiedy mamy limit zdjęć do importu, a tylko one mogą być używane w opisie aukcji? Jak wstawić tabele rozmiarów (edit: allegro ma w planie wprowadzenie możliwości wstawiania tabel a nawet video do opisu aukcji), czy kilka galerii? 

Słowem: "Lypa". Najlogiczniejsze wydaje się wystawienie każdego wariantu osobno i połączenie ich przez dostępny dla danej kategorii parametr - czyli stworzenie wielowariantowości. 

*Dygresja: z jednej aukcji nagle powstaje 10-20, to dlatego lista aukcji w kategorii ma po milion stron, i dlatego zajmie ci wieki by przebrnąć przez aukcje jednego sprzedawcy, który wystawił właśnie po 20 wariantów każdego buta na pojedyńczych aukcjach.*

Znowu kłoda, Zespół Allegro nie pomyślał, aby udostępnić cokolwiek, zasób w WebAPI czy w REST, co mogłoby pomóc w automatyzacji łączenia aukcji. Zamiast tego, poleca klikanie. 

<figure class="center">
	<img src='{{ site.url }}/images/allegro/rant.jpg' alt="">
	<figcaption>źródło: Oficjalny fanpage Allegro API na facebook</figcaption>
</figure>

<figure class="center">
	<img src='{{ site.url }}/images/allegro/rant2.jpg' alt="">
	<figcaption>Jak długo jeszcze czekać będziemy na NIEZBĘDNE narzędzia?</figcaption>
</figure>

Brrr... aż mi ciarki po plecach przeszły na myśl o ręcznym łączeniu 10000 aukcji. Więc, dwie kawy później, przedstawiam koncepcję i sposób jak można to częściowo zautomatyzować [w tym poście]({{ site.baseurl }}{% post_url /ecommerce/allegro/2018-02-14-allegro-pl-automatyczne-grupowanie-aukcji-wielowariantowosc %})

Podsumowując ten mały *rant*, monopol Allegro.pl na rynku serwisów aukcyjnych zmusza wielu by się dostosowali albo "niech giną", a łatwo nie jest. W kolejnych wpisach szerzej omawiać będę jak korzystać z zasobów starego WebAPI (co nie jest takie proste), oraz zasobów REST API. Będą gotowe skrypty dla ułatwiania sobie procesu wystawiania aukcji na nowych zasadach. Zachęcam do odwiedzania mojego bloga.
