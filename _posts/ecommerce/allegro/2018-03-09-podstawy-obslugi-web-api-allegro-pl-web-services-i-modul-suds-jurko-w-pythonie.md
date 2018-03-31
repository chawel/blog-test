---
layout: post
title: Podstawy obsługi Web API Allegro.pl - Web Services i moduł suds-jurko w Pythonie
modified:
categories: ecommerce/allegro
description: Wprowadzenie do obsługi WebAPI - protokół SOAP, WSDL oraz moduł suds
tags: [allegro, web services, suds, python]
image:
  feature: allegro_webapi_schemat.png
  credit: Dokumentacja WebAPI Allegro
  creditlink: https://allegro.pl/webapi/general.php
comments:
share:
date: 2018-03-09T17:06:20+01:00
---

Dzisiaj o tym jak ożenić Pythona z Web API udostępnionym przez Allegro.pl. Jest to pierwszy, wprowadzający do tematu wpis z nadchodzącej serii. Na wstępie chciałbym zaznaczyć, że nie będę wgłębiał się w teorię, terminologię ani zasady działania Web Services, czy SOAP, WSDL, XML, Łosie, Jelenie, Sarny, Dziki, Kuny. Wymienione elementy postaram się ominąć jak najszerszym łukiem i tylko niezbędne pobieżnie nakreślić. 

<!-- more -->

Ktoś może z oburzeniem spytać: *"To jak to tak, niby wpis o tym jak używać Web Services a nic nie będziesz tłumaczył?!"*, już odpowiadam: *"Tak, w sensie, że nie będę tłumaczył"*. Skupie się jedynie na praktyce, ponieważ cały temat Web Services jest tak obszerny, że aż się odechciewa pisać ten wpis. No, ale nie przejmuj się czytelniku! Zapewniam, że do korzystania z zasobów Web API serwisu Allegro.pl nie potrzebujesz tej wiedzy.

Tych, którzy jednak liczyli na jakieś wyjaśnienia jak Web API u Allegro działa, kieruje do dokumentacji, gdzie jest to całkiem nieźle wyjaśnione (więc po co powtarzać): [WebAPI w pigułce - Oficjalna dokumentacja](https://allegro.pl/webapi/general.php)

Ten wpis poświęcę na omówienie samej komunikacji z Web Serwisem, o tym jak to zrobić przy użyciu Python i biblioteki *suds-jurko* (fork oryginalnej biblioteki *suds*). Jak zwykle będą przykłady banalne i *"po najmniejszej linii oporu"*, bez przejmowania się o wydajność i bezpieczeństwo, dlatego jeżeli piszesz "dla siebie" - to spoko, ale jak chcesz komuś opchnąć jakiś skrypt - to poczekaj do następnej części gdzie będzie o REST API Allegro (bezpieczniejsze logowanie).


## Uzyskanie klucza Web API.
Zanim zaczniemy, musimy uzyskać klucz Web API z panelu naszego konta na Allegro.pl. Służy on do identyfikacji podczas komunikacji, ale nie jest to uwierzytelnienie (od tego będzie id sesji).

Logujemy się na nasze konto w Allegro.pl, następnie przechodzimy do zakładki *Moje konto*, tam na dole na ekranie głównym odnajdujemy sekcję *WebAPI*, a w niej odnośnik [Informacje i ustawienia](https://allegro.pl/myaccount/webapi.php)

<figure class="center">
	<img src='{{ site.url }}/images/allegro/allegro_mojeallegro.png' alt="">
	<figcaption>Moje allegro</figcaption>
</figure>

Tam czeka na nas wirtualny policjant, który musi wiedzieć, czym motywujemy prośbę o wygenerowanie naszego klucza. Odpowiadamy zgodnie z własnym sumieniem, pamiętając o tym, że wszystko co napiszemy może zostać użyte przeciwko nam... (żart)

<figure class="center">
	<img src='{{ site.url }}/images/allegro/allegro_webapikey_gen.png' alt="">
	<figcaption>Ktoś to w ogóle czyta? Po co te wyjaśnienia?</figcaption>
</figure>

Zgarniamy wygenerowany klucz i jedziemy dalej.

<figure class="center">
	<img src='{{ site.url }}/images/allegro/allegro_webapikey.png' alt="">
	<figcaption>Klucz Allegro WebAPI</figcaption>
</figure>

## Obsługa protokołu SOAP w Python

[SOAP](https://www.w3schools.com/XML/xml_soap.asp) - czyli protokół komunikacji naszego zainteresowania, w uproszczeniu polega na wysyłaniu i odbieraniu żądań w postaci składni XML.

Potrzebny będzie tytułowy moduł [suds](https://pypi.python.org/pypi/suds), ale jest on przestarzały i od dawna nieaktualizowany, więc proponuję trochę bardziej aktualny [fork](https://pl.wikipedia.org/wiki/Fork) tego modułu o nazwie [suds-jurko](https://bitbucket.org/jurko/suds). Jest jeszcze biblioteka [Zeep](http://docs.python-zeep.org/en/master/), ale nie miałem z nią przyjemności, dlatego użyję sprawdzonej metody.

{% highlight python %}
from suds.client import Client
{% endhighlight %}

[WSDL](https://www.w3schools.com/XML/xml_wsdl.asp) - jest to język pliku który definiuje dany Web Service, z niego dowiadujemy się co ma on nam do zaoferowania (zasoby), jakie typy obsługuje, stuktury danych wymienianych, no ogólnie taka w uproszczeniu dokumentacja w XML z której się dowiadujemy co on zaś i jak z nim gadać.

Musimy jakoś przyswoić i przerobić WSDL, żeby wiedzieć jak z danym Web Service gadać. Moduł *suds* ma na to prosty sposób, importujemy klasę `Client` z jego zasobów a następnie inicjujemy ją, przekazując w jej parametrze URL do WSDL danego Web Api, tutaj wiadomo, chodzi nam o te dla Allegro, czyli: 

``` https://webapi.allegro.pl/service.php?wsdl ```

Możecie zrobić mały test, żeby bardziej zrozumieć czym WSDL jest, wystarczy wklepać powyższy URL jako adres do przeglądarki. Wyświetlić się powinna struktura w formie XML, można rozejrzeć się i popatrzeć co jest w środku.

{% highlight python %}
client = Client('https://webapi.allegro.pl/service.php?wsdl')
{% endhighlight %}

Mamy instancję klienta, który będzie się komunikował z Web API. Aby przesłać jakieś żądanie, musimy odwołać się do odpowiedniego zasobu w serwisie. Spokojnie, to nic trudnego, wszystkie zasoby opisane są w [oficjalnej dokumentacji](https://allegro.pl/webapi/documentation.php). Odwołajmy się więc do zasobu [doGetSystemTime](https://allegro.pl/webapi/documentation.php/show/id,81), którego opis to:
> "Metoda pozwala na pobranie aktualnego (dla danego kraju) czasu z serwera Allegro".

Metoda przyjmuje parametry wejściowe, które muszą odpowiadać temu, co spodziewa się otrzymać (zgodny typ, wymagane pola itd.). W tym przypadku, z dokumentacji (ale również z pliku WSDL) możemy poznać co konkretnie musimy dołączyć: 

```
countryId | int | wymagany | Identyfikator kraju (listę identyfikatorów krajów uzyskać można za pomocą metody doGetCountries).
webapiKey | string | wymagany | Klucz WebAPI użytkownika.
```

Przykład wywołania:
{% highlight python %}
response = client.service.doGetSystemTime(webapiKey='<nasz_klucz_webapi>', countryId=1)
{% endhighlight %}

Wynik:
```
>>> print(response)
1269807050
```

Teraz coś, co zwróci nam trochę bardziej rozbudowaną odpowiedź (uzyskanie klucza wersji API dla Polski, przyda się w dalszej części):
{% highlight python %}
response = client.service.doQueryAllSysStatus(webapiKey='<nasz_klucz_webapi>', countryId=1)
{% endhighlight %}

Wynik:
```
>>> print(response)
(ArrayOfSysstatustype){
   item[] = 
      (SysStatusType){
         countryId = 1
         programVersion = "1.0"
         catsVersion = "1.6.41"
         apiVersion = "1.0"
         attribVersion = "1.0"
         formSellVersion = "1.14.65"
         siteVersion = "1.0"
         verKey = 1520603076
      },
 }
```

Jak widzimy, zwrócona struktura przypomina trochę znany nam typ dict. Niestety, nie jest to dict, musimy się trochę wysilić aby wyłuskać elementy które nas interesują. Dodam tylko, że zwrócone dane nie powinny być dla nas zaskoczeniem, ponieważ jak wcześniej wspominałem, wszystkie typy, struktury danych (przyjmowanych i zwracanych) są zdefiniowane w pliku WSDL (jak i w dokumentacji).

Widzimy że typy są określone w nawiasach, również jest hierarchiczna struktura danych. Klient wstępnie zadbał o przekonwertowanie danych, dla naszej wygody. Na górze (w hierarchi) jest lista `item[]` a w niej zawarte dane. Spróbujmy ją wyświetlić.

```
>>> print(response.item)
[(SysStatusType){
   countryId = 1
   programVersion = "1.0"
   catsVersion = "1.6.41"
   apiVersion = "1.0"
   attribVersion = "1.0"
   formSellVersion = "1.14.65"
   siteVersion = "1.0"
   verKey = 1520603076
 }]
```

Ok, działa, możemy więc w łatwy sposób uzyskać dane ze struktury. Wejdźmy jeszcze głębiej.
```
>>> print(response.item[0].verKey)
1520603076
```

Skoro `item[]` to lista - to aby odwołać się do jej zawartości, podajemy index obiektu który nas interesuje. W tym przypadku jest tylko jeden, więc ma index o numerze 0. Dalej, z tego obiektu wyciągamy `verKey`.

## Uwierzytelnienie - czyli logowanie
Uzyskaliśmy fundamentalną wiedzę jak porozumiewać się z Web API, możemy więc przejść do meritum, czyli jak uwierzytelnić nasze poczynania, a co za tym idzie uzyskać dostęp do zasobów, które tego wymagają. Musimy uzyskać `sessionId` (znany również jako `sessionHandler`).

#### UWAGA! Jest to bardzo niebezpieczna metoda! Przekazujemy dane do NASZEGO KONTA na Allegro w formie plain text!
{% highlight python %}
auth = client.service.doLogin(
    userLogin='<login>',
    userHashPassword='<haslo>', 
    countryCode=1,
    webapiKey='<nasz_klucz_webapi>', 
    localVersion='<uzyskany_wczesniej_verKey>'
)
{% endhighlight %}

```
>>> print(auth.sessionHandlePart)
22eb99326c6be29aa16d07d622bcfbcbee94ad54846f2f4e03
```

To jest nasz identyfikator sesji, który przekazywać będziemy w parametrach `sessionId` lub `sessionHandler` dla zasobów wymagających uwierzytelnienia.

#### UWAGA! Jest inny zasób, który jest tak samo niebezpieczny, ale zapewnia "ochronę" naszego hasła, ponieważ je hashuje. W przypadku wykradnięcia hasha, będzie można przy jego użyciu się zalogować przez Web API, ale chroni to ludzi, którzy mają wszędzie to samo hasło (chociaż też nie koniecznie - ale nie o tym teraz).

{% highlight python %}
import hashlib
import base64

auth = client.service.doLoginEnc(
    userLogin='<login>',
    userHashPassword=base64.b64encode(hashlib.sha256('<haslo>'.encode('utf-8')).digest()).decode('utf-8'), 
    countryCode=1,
    webapiKey='<nasz_klucz_webapi>', 
    localVersion='<uzyskany_wczesniej_verKey>'
)
{% endhighlight %}

## Cały przykładowy kod
{% highlight python %}
from suds.client import Client
import hashlib
import base64

client = Client('https://webapi.allegro.pl/service.php?wsdl')

response = client.service.doQueryAllSysStatus(webapiKey='<nasz_klucz_webapi>', countryId=1)

version_key = response.item[0].verKey

auth = client.service.doLoginEnc(
    userLogin='<login>',
    userHashPassword=base64.b64encode(hashlib.sha256('<haslo>'.encode('utf-8')).digest()).decode('utf-8'), 
    countryCode=1,
    webapiKey='<nasz_klucz_webapi>', 
    localVersion='<uzyskany_wczesniej_verKey>'
)
{% endhighlight %}

Oczywiście, nie oznacza to, że samo użycie tych metod i przekazaniu przez internet hasła i loginu w *plain text*, spowoduje automatyczne przejęcie ich przez osoby niepowołane, ale jest takowe ryzyko. Mimo użycia protokołu *HTTPS*, czyli połączenia szyfrowanego, musimy być ostrożni. Najlepiej, gdy używamy tej metody, zmienić hasło na unikalne (nie powtarzające się nigdzie indziej, np. do skrzynki e-mail, ftp, panelu sklepu internetowego) i zmienić je od razu po skończonej pracy takiego skryptu. Jeżeli chcemy wykorzystać skrypt, np. do automatyzacji i będzie on umieszczony na zewnętrznym serwerze, odpalany przez CRON, lub w inny bezobsługowy i powtarzający się sposób, proponuję skorzystać z możliwości uzyskania `token` przez najnowsze REST API Allegro i przekazania go do zasobu [doLoginWithAccessToken](https://allegro.pl/webapi/documentation.php/show/id,101582). Cały proces logowania z użyciem REST API, oraz jak obsługiwać 2 API na raz w jednym skrypcie opiszę już wkrótce w kolejnych wpisach.

## UPDATE! Bezpieczniejsze logowanie do WebAPI
Ważna informacja, która ratuje nas z opresji, którą było super głupie logowanie się do WebAPI przy użyciu hasła i loginu do konta allegro (w dodatku w *plain text*)! Co więcej, zespół Allegro wprowadził [dwustopniowe logowanie](https://allegro.pl/dla-sprzedajacych/dwustopniowe-logowanie-D5VXVldVbHo) do naszego konta w serwisie Allegro.pl! Jest to niezwykle dobra informacja, dla ludzi którzy cenią sobie wyłączność na dostęp do swojego konta. Co z WebAPI? Również sytuacja się poprawiła! Wprowadzono [hasła do aplikacji](https://allegro.pl/myaccount/Settings/security_settings.php/applicationPasswords), które chronią nas na wypadek wycieku hasła używanego do uwierzytelnienia naszych skryptów w WebAPI (bo jest różne od tego do konta).

Co więcej, gdy ustawimy *dwustopniowe logowanie* do naszego konta, **musimy** użyć zdefiniowanego *hasła do aplikacji* - nie będzie możliwości zalogowania się przy użyciu danych do konta.

<figure class="center">
	<img src='{{ site.url }}/images/allegro/allegro_app_passwd.gif' alt="">
	<figcaption>Moje allegro > Bezpieczeństwo > Hasła do aplikacji</figcaption>
</figure>

To na szybko, jak w **bezpieczny** sposób uzyskać `sessionId` używając *hasła aplikacji*.
{% highlight python %}
from suds.client import Client
import hashlib
import base64

client = Client('https://webapi.allegro.pl/service.php?wsdl')

response = client.service.doQueryAllSysStatus(webapiKey='<nasz_klucz_webapi>', countryId=1)

version_key = response.item[0].verKey

sha256_application_password = hashlib.sha256('<haslo_do_aplikacji>'.encode('utf-8')).digest()

auth = client.service.doLoginEnc(
    userLogin='<login>',
    userHashPassword=base64.b64encode(sha256_application_password).decode('utf-8'), 
    countryCode=1,
    webapiKey='<nasz_klucz_webapi>', 
    localVersion=version_key
)

session = auth.sessionHandlePart
{% endhighlight %}

Niestety, nie zmienia to faktu, że nadal będę musiał napisać o tym jak żonglować dwoma API na raz, bo tak jak zdejmuje to ciężar autoryzacji w WebAPI przez REST API, to i tak nadal potrzebujemy REST API do procesu wystawiania aukcji (co zmniejsza jeszcze bardziej jego użyteczność na chwilę obecną).

