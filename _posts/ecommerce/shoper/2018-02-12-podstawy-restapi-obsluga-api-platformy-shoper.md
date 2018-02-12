---
layout: post
title: Podstawy REST API - Obsługa API platformy Shoper
modified:
categories: ecommerce/shoper
description: Wstęp do automatyzacji Shoper, czyli Python (Requests) + REST Api w akcji.
tags: [shoper, rest, python]
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2018-02-12T18:14:02+01:00
---

Jak szybko i w najprostszy sposób zacząć korzystać z *dobrodziejstw* wystawionych przez REST? Pokażę jak niewiele linijek kodu potrzeba aby postawić pierwszy krok w stronę automatyzacji obsługi Twojego sklepu. Python i jego świetna biblioteka Requests w akcji.

<!-- more -->

## Wstęp

W tym poście postaram się, w miare zwięźle i bez zbędnych dygresji, pokazać jak odwoływać się do zasobów REST Api platformy Shoper, używając do tego zadania biblioteki Requests. Z uwagi na to, że ma być zwięźle, niestety nie będę omawiać biblioteki Requests, tak jak i samej koncepcji oraz zasad działania RESTowych serwisów. Jeżeli więc nie masz zielonego pojęcia o tym czym jest POST, GET, Token, nie wiesz do czego służy REST, warto takie informacje pozyskać (postaram się w niedalekiej przyszłości uzupełnić blog o wpisy wyjaśniające te zagdnienia). Teraz skupie się na totalnej praktyce i o tym jak zacząć przy *minimum* wiedzy i bez wysiłku (co nie oznacza, że jest to najlepszy sposób, powiem więcej, raczej jest to sposób dla osób, które muszą coś zrobić na wczoraj albo wykonują bardzo proste skrypty dla SIEBIE).

## Uzyskanie dostępu do zasobów (uwierzytelnienie klienta)

### Shoper - panel administracyjny

#### Zacznijmy od stworzenia konta, z którego korzystać będzie nasz skrypt.

W menu wybieramy *Konfiguracja > Administracja, system*

<figure class="center">
	<img src='{{ site.url }}/images/shoper/shoper_menu_adminconf.gif' alt="">
	<figcaption>Konfiguracja > Administracja, system</figcaption>
</figure>

Możemy stworzyć nową grupę administratorów (np. nazywająć ją *WebApi*). Wybieramy w prawym górnym rogu opcje *Dodaj grupę administratorów*

<figure class="center">
	<img src='{{ site.url }}/images/shoper/shoper_admingroup.gif' alt="">
	<figcaption>Tworzenie grupy administratorów.</figcaption>
</figure>

Tworzymy nowego administratora. Wybieramy w prawym górnym rogu opcje *Dodaj administratora*

<figure class="center">
	<img src='{{ site.url }}/images/shoper/shoper_admin.gif' alt="">
	<figcaption>Tworzenie administratora.</figcaption>
</figure>

<figure class="center">
	<img src='{{ site.url }}/images/shoper/shoper_admin_form.gif' alt="">
	<figcaption>Formularz tworzenia administratora.</figcaption>
</figure>

Nadajemy odpowiednie uprawnienia grupie w której znajduje się nasz admin - proponuję nie przesadzać, zawsze lepiej coś dodać w miare potrzeb, bo jak wiemy 
> "WITH GREAT POWER THERE MUST ALSO COME--GREAT RESPONSIBILITY!"

Klikamy na *nazwa grupy > Uprawnienia*
<figure class="center">
	<img src='{{ site.url }}/images/shoper/shoper_admingroup_prem.gif' alt="">
	<figcaption>Uprawnienia.</figcaption>
</figure>

Ok to na tyle co potrzebujemy z panelu administracyjnego.

### Python

Importujemy bibliotekę [Requests](http://docs.python-requests.org/en/latest/)

{% highlight python %}
import requests
{% endhighlight %}

Kolejnym krokiem będzie zainicjowanie sesji, która ułatwi nam robotę (będzie przechowywać parametry przekazywane razem z żądaniami, jak np. nagłówki, ciastka).

{% highlight python %}
s = requests.Session()
{% endhighlight %}

Jesteśmy gotowi do uzyskania Token'a (w uproszczeniu jest to kod przekazywany w nagłówku, aby serwis REST mógł nas rozpoznać, i udzielić dostępu do zasobów). Aby taki token uzyskać, musimy metodą POST przekazać login i hasło do konta, które stworzyliśmy w panelu administracyjnym sklepu.

{% highlight python %}
response = s.post('https://<url_naszego_sklepu>/webapi/rest/auth', auth=(<login>, <haslo>))
{% endhighlight %}

Jeżeli login, hasło i adres naszego sklepu się zgadza, powinniśmy otrzymać odpowiedź, którą przypisaliśmy do zmiennej *response*. Odpowiedzi zwykle są opisane w składni JSON, ale metoda post() zwraca nam obiekt. Wypisując *response*, otrzymamy jedynie status odpowiedzi.

{% highlight python %}
>> print(response)
<Response [200]>
{% endhighlight %}

Aby dobrać się do odpowiedzi z tokenem, musimy podejrzeć co konretnie zwrócił nam REST.

{% highlight python %}
>> print(response.json())
{'access_token': 'xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx', 'expires_in': 2592000, 'token_type': 'bearer'}
{% endhighlight %}

Metoda json() zwróci strukturę, z której w prosty sposób można wyłuskać *access_token*. Cała procedura będzie wyglądać tak:

{% highlight python %}
response = s.post('https://<url_naszego_sklepu>/webapi/rest/auth', auth=(<login>, <haslo>))
result = response.json()
token = result['access_token']
{% endhighlight %}

Uzyskaliśmy *access_token* i przypisaliśmy go do zmiennej *token*. Wypada teraz go użyć. Jak pisałem, *access_token* musimy przekazywać jako element nagłówka każdego żądania, do tego własnie przyda się nam sesja, aby nie musieć pamiętać o jego przekazywaniu. Dodajemy go po prostu jako nagłówek sesji.

{% highlight python %}
s.headers.update(headers)
{% endhighlight %}

Od tej pory nasze żądania są uwierzytelnione, co oznacza, że możemy swobodnie korzystać z zasobów REST Api.

### Przykładowe odwołanie się do zasobu

Do dyspozycji, mamy operacje [CRUD](https://pl.wikipedia.org/wiki/CRUD). Wszystkie zasoby (opisane) znajdziemy w [Dokumentacja REST API Shoper](https://developers.shoper.pl/developers/api/getting-started). Tak więc jak to mówią: "The sky is the limit". Przykład uzyskania listy produktów:

{% highlight python %}
response = s.get(URL + '/products')
print(response.json())
{% endhighlight %}

Warto zauważyć, że w odpowiedzi dostajemy tylko jedną ze stron, zawierającą określoną liczbę rekordów (kolekcje). Na tym zakończę ten post, ponieważ to, jak po kolei "przejść" po tych stronach (stronicowanie - czy jak coraz częściej się spotyka, paginacja), wymaga wiedzy jak dodawać do naszych rządań parametry, a o tym w następnej części o REST Api.




