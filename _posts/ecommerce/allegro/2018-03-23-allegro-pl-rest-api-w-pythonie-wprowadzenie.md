---
layout: post
title: Allegro.pl REST API w Pythonie - wprowadzenie (OAuth)
modified:
categories: ecommerce/allegro
description: Wprowadzenie do obsługi REST API, OAuth 2 - czyli uwierzytelnianie.
tags: [allegro, rest, python]
image:
  feature: rest_allegro.jpg
  credit:
  creditlink:
comments:
share:
date: 2018-03-23T23:26:17+01:00
---

Wpis o tym jak zacząć korzystać z REST API serwisu Allegro.pl w skryptach pisanych w Pythonie. Postaram się wyjaśnić niezbędne minimum, do tego by zacząć od razu korzystać z API.

<!-- more -->

Jeżeli czytałeś poprzedni wpis [Podstawy REST API - Obsługa API platformy Shoper]({{ site.baseurl }}{% post_url ecommerce/shoper/2018-02-12-podstawy-restapi-obsluga-api-platformy-shoper %})
 to prawdopodobnie kilka razy będziesz miał wrażenie *deja vu*, ale nic dziwnego, skoro znowu omawiamy serwis o architekturze REST. Standardowo, nie będę tłumaczył zasad działania tego typu serwisu, ani nie będę wnikał w poszczególne metody HTTP do jego obsługi. Zainteresowanych teorią odsyłam do [Oficjalnej Dokumentacji](https://developer.allegro.pl/about/), gdzie jest to całkiem nieźle wyjaśnione.

## Wstęp
Na początek, musimy wykonać pewne czynności aby uzyskać dostęp do REST API, czyli zarejestrować naszą aplikację (lub skrypt jak kto woli), uzyskując przy tym dane dostępowe do REST API potrzebne do uzyskania *token*.

### Istotna uwaga: 
**Na chwilę obecną nie możemy "odwiązać" aplikacji, inaczej mówiąc, raz podwiązana aplikacja do konta - zawsze będzie miała dostęp do API. Tak samo z wygenerowanymi danymi dostępowymi - nie można ich zdjąć czy odwołać dostępu za ich pomocą - dlatego lepiej jest nie umieszczać ich w kodzie źródłowym programu czy chwalić się nimi.**


Pełna instrukcja jak działa rejestracja aplikacji oraz cała procedura opisana została w [Oficjalnej Dokumentacji - Rejestracja Aplikacji](https://developer.allegro.pl/auth/).

Dla osób z *tekstowstrętem*, cały proces w obrazkach.
{% capture images %}
	/images/allegro/allegro_rest_reg1.png
	/images/allegro/allegro_rest_reg2.png
	/images/allegro/allegro_rest_reg3.png
{% endcapture %}
{% include gallery images=images caption="Proces rejestracji aplikacji" cols=2 %}

## Uzyskanie tokena (uwierzytelnienie aplikacji)
Będziemy brali udział w zabawie zwanej [OAuth 2.0](https://oauth.net/2/) - polega ona na wykonaniu kilku czynności, aby udzielić naszej aplikacji uprawnień do wykonywania operacji w imieniu naszego konta na allegro. Po kolei:

1. Rejestrujemy naszą aplikację (to już za nami) i uzyskanie danych dostępowych
2. Wysyłamy żądanie do odpowiedniego endpointa przekazując dane dostępowe oraz tzw. *redirect uri* (o nim za chwilę)
3. W tym momencie w graficznym interfejsie (zwykle dzięki otwarciu się przeglądarki) zostaniemy przekierowani do formularza logowania do naszego konta na Allegro.pl
4. Wyrażamy zgodę na powiązanie aplikacji z naszym kontem na Allegro.pl
5. Teraz, na adres podany w *redirect_uri* zostanie wysłany *kod autoryzacyjny* (jest on **jednorazowy i ważny 10 sekund!**)
6. Wysyłamy kolejne żądanie na odpowiedni adres, w którego ciele przekazujemy odpowiednią zawartość, w tym wcześniej uzyskany *kod autoryzacyjny*.
7. Zwrócona odpowiedź (w formie JSON) powinna zawierać nasz *token* - czyli ciąg znaków używany do uwierzytelniania naszych poczynań przez REST API.

Jest trochę tego, więc od razu zaczynamy implementację - kod omówiony w komentarzach.

{% highlight python %}
import requests
from http.server import BaseHTTPRequestHandler, HTTPServer
import webbrowser

# Dla ułatwienia, definiujemy domyślne wartości (tak zwane stałe), są one uniwersalne
DEFAULT_OAUTH_URL = 'https://allegro.pl/auth/oauth'
DEFAULT_REDIRECT_URI = 'http://localhost:8000'

# Implementujemy funkcję, której parametry przyjmują kolejno:
#  - client_id (ClientID), api_key (API Key) oraz opcjonalnie redirect_uri i oauth_url
# (jeżeli ich nie podamy, zostaną użyte domyślne zdefiniowane wyżej)
def get_access_code(client_id, api_key, redirect_uri=DEFAULT_REDIRECT_URI, oauth_url=DEFAULT_OAUTH_URL):
    # zmienna auth_url zawierać będzie zbudowany na podstawie podanych parametrów URL do zdobycia kodu
    auth_url = '{}/authorize' \
               '?response_type=code' \
               '&client_id={}' \
               '&api-key={}' \
               '&redirect_uri={}'.format(oauth_url, client_id, api_key, redirect_uri)
 
    # uzywamy narzędzia z modułu requests - urlparse - służy do spardowania podanego url 
    # (oddzieli hostname od portu)
    parsed_redirect_uri = requests.utils.urlparse(redirect_uri)

    # definiujemy nasz serwer - który obsłuży odpowiedź allegro (redirect_uri)
    server_address = parsed_redirect_uri.hostname, parsed_redirect_uri.port

    # Ta klasa pomoże obsłużyć zdarzenie GET na naszym lokalnym serwerze
    # - odbierze żądanie (odpowiedź) z serwisu allegro
    class AllegroAuthHandler(BaseHTTPRequestHandler):
        def __init__(self, request, address, server):
            super().__init__(request, address, server)

        def do_GET(self):
            self.send_response(200, 'OK')
            self.send_header('Content-Type', 'text/html')
            self.end_headers()

            self.server.path = self.path
            self.server.access_code = self.path.rsplit('?code=', 1)[-1]

    # Wyświetli nam adres uruchomionego lokalnego serwera
    print('server_address:', server_address)

    # Uruchamiamy przeglądarkę, przechodząc na adres zdefiniowany do uzyskania kodu dostępu
    # wyświetlić się powinien formularz logowania do serwisu Allegro.pl
    webbrowser.open(auth_url)

    # Uruchamiamy nasz lokalny web server na maszynie na której uruchomiony zostanie skrypt
    # taki serwer dostępny będzie pod adresem http://localhost:8000 (server_address)
    httpd = HTTPServer(server_address, AllegroAuthHandler)
    print('Waiting for response with access_code from Allegro.pl (user authorization in progress)...')

    # Oczekujemy tylko jednego żądania
    httpd.handle_request()

    # Po jego otrzymaniu zamykamy nasz serwer (nie obsługujemy już żadnych żądań)
    httpd.server_close()

    # Klasa HTTPServer przechowuje teraz nasz access_code - wyciągamy go
    _access_code = httpd.access_code

    # Dla jasności co się dzieje - wyświetlamy go na ekranie
    print('Got an authorize code: ', _access_code)

    # i zwracamy jako rezultat działania naszej funkcji
    return _access_code
{% endhighlight %}

W czasie działania tej funkcji, powinna otworzyć się nam przeglądarka z formularzem logowania do serwisu Allegro.pl, logujemy się, następnie wyświetli się komunikat z zapytaniem, czy wyrażamy zgodę na powiązanie aplikacji która o to prosi - zgadzamy się. **(pkt. 3-4)**

{% capture images %}
	/images/allegro/allegro_rest_auth1.png
	/images/allegro/allegro_rest_auth2.png
{% endcapture %}
{% include gallery images=images caption="Proces autentykacji aplikacji" cols=2 %}

Wynikiem tej operacji powinno być uzyskanie *kodu autoryzacyjnego* **(pkt. 5)**
```
>> access_code = get_access_code('<client_id>', '<api_key>')
server_address: ('localhost', 8000)
Waiting for response with access_code from Allegro.pl (user authorization in progress)...
127.0.0.1 - - [30/Mar/2018 12:42:06] "GET /?code=xxxxxxxxxxxxxxxxxxxxx HTTP/1.1" 200 -
Got an authorize code:  xxxxxxxxxxxxxxxxxxxxx
```

**W ciągu 10 sekund** musimy przekazać `access_code` w żądaniu typu *POST* na adres HTTP `https://allegro.pl/auth/oauth/token` z odpowiednim nagłówkiem zawierającym `client_id` oraz `client_secret` (oba otrzymaliśmy przy rejestracji naszej aplikacji) w formie zakodowanej Base64. **(pkt. 6)**
{% highlight python %}
def sign_in(client_id, client_secret, access_code, api_key, redirect_uri=DEFAULT_REDIRECT_URI, oauth_url=DEFAULT_OAUTH_URL):
    token_url = oauth_url + '/token'

    access_token_data = {'grant_type': 'authorization_code',
                         'code': access_code,
                         'api-key': api_key,
                         'redirect_uri': redirect_uri}

    response = requests.post(url=token_url,
                             auth=requests.auth.HTTPBasicAuth(client_id, client_secret),
                             data=access_token_data)

    return response.json()
{% endhighlight %}

I na koniec otrzymujemy odpowiedź w formie *JSON*, który zawiera `access_token` oraz `refresh_token`. Możemy zapisać oba **(byle w bezpieczny sposób!)** w pliku, bazie, na kartce nie polecam - i korzystać już bezpośrednio z nich nie powtarzając całego procesu. 

Czas ważności `access_token` określony jest w `expires_in` (w sekundach - będzie to 12 godzin), po tym czasie należy wykorzystać `refresh_token` aby uzyskać nowy zestaw zawierający nowy `access_token` (oraz nowy `refresh_token`), który to znowu ważny jest następne 12 godzin. Można tak przedłużać przez następne 365 dni, po tym czasie będziemy musieli znowu zalogować się do allegro przez formularz (czyli powtórzyć cały proces opisany wyżej).
```
>> print(sign_in('<client_id>', '<client_secret>', access_code, '<api_key>'))
{'access_token': 'xxxxxx',
'token_type': 'bearer',
'refresh_token': 'xxxxxxxxxx',
'expires_in': 43199,
'scope': 'allegro_api',
'jti': 'xxxx-xxxx-xxxx-xxxx-xxx'}
```

## Przedłużanie ważności tokena
Kiedy minie 12 godzin od uzyskania `access_token`, staje się on nieważny i nie możemy z jego pomocą uwierzytelnić naszej aplikacji. Należy skorzystać z `refresh_token` do jego odświeżenia.
{% highlight python %}
def refresh_token(client_id, client_secret, refresh_token, api_key, redirect_uri=DEFAULT_REDIRECT_URI, oauth_url=DEFAULT_OAUTH_URL):
    token_url = oauth_url + '/token'

    access_token_data = {'grant_type': 'refresh_token',
                         'api-key':  api_key,
                         'refresh_token': refresh_token,
                         'redirect_uri': redirect_uri}

    response = requests.post(url=token_url,
                             auth=requests.auth.HTTPBasicAuth(client_id, client_secret),
                             data=access_token_data)

    return response.json()
{% endhighlight %}

Przykładowe wywołanie
```
>> print(refresh_token('<client_id>', '<client_secret>', '<refresh_token>', '<api_key>'))
{'access_token': 'xxxxxx',
'token_type': 'bearer',
'refresh_token': 'xxxxxxxxxx',
'expires_in': 43199,
'scope': 'allegro_api',
'jti': 'xxxx-xxxx-xxxx-xxxx-xxx'}
```

I tak w koło przez kolejne 365 dni.

## Korzystanie z zasobów REST API

Nie chcę dodawać zbyt wiele do i tak już obszernego wpisu, dlatego pokażę jedynie działający koncept jak można wykorzystać uzyskany `access_token` do uwierzytelnienia żądań wysyłanych do API. Chcę poświęcić zagadnieniu żądań HTTP (POST, GET, PUT itd.) osobny wpis. W przedstawionym przykładzie uzyskujemy *ID* karty gwarancji, zdefiniowanej w ustawieniach sprzedaży (jest to element wymagany do wystawienia aukcji z nowym opisem). Odwołując się do zasobu `/after-sales-service-conditions/warranties` musimy przekazać metodą *GET* również parametr `sellerId`. 

Wyjaśnię tylko czym jest `sellerId` - jest to *identyfikator* naszego konta, można go uzyskać przez *WebAPI*, ale łatwiej jest po prostu wejść w [Moja karta użytkownika](https://allegro.pl/uzytkownik/oceny), odnośnik do niej (URL) zawiera właśnie nasze `sellerId`

`https://allegro.pl/uzytkownik/<seller_Id>/oceny`

{% highlight python %}
# Budujemy nagłówek naszego żądania (wymagany)
headers = {}
headers['charset'] = 'utf-8'
headers['Accept-Language'] = 'pl-PL'
headers['Content-Type'] = 'application/json'
headers['Api-Key'] = api_key
headers['Accept'] = 'application/vnd.allegro.public.v1+json'
headers['Authorization'] = "Bearer {}".format(access_token)

# Inicjujemy naszą sesję (przechowuje nagłówki itd.)
# konstrukcja with pozwala na użycie sesji tylko w jej obrębie
# kiedy wyczerpią się instrukcje wewnątrz niej
# straci ona ważność (zostanie zamknięta)
with requests.Session() as session:
    session.headers.update(headers)

    response = session.get(DEFAULT_API_URL + '/after-sales-service-conditions/warranties', 
                           params={'sellerId':'<nasz_sellerId>'})
    
    # Wypisz odpowiedź w formacie JSON
    print(response.json())

# Dla przykładu: tutaj już opuściliśmy nasz 'with'
print("Koniec programu")
# umieszczając w tym miejscu
# np. print(session) otrzymalibyśmy błąd - ponieważ już nie istnieje.

{% endhighlight %}
