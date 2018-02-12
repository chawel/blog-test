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

Jak szybko i w najprostszy sposób skorzystać z *dobrodziejstw* wystawionych przez REST? Pokażę jak niewiele linijek kodu potrzeba, aby postawić pierwszy krok w stronę automatyzacji obsługi Twojego sklepu. Python i jego świetna biblioteka Requests w akcji.

<!-- more -->

## Wstęp

W tym poście postaram się, w miare zwięźle i bez zbędnych dygresji, zaprezentować jak odwoływać się do zasobów REST API platformy Shoper. Użyję do tego zadania biblioteki Requests. Z uwagi na to, że ma być zwięźle, nie będę omawiać biblioteki Requests, jak również koncepcji i zasad działania RESTowych serwisów. Jeżeli więc nie masz zielonego pojęcia o tym czym są POST, GET, Token, nic Ci nie mówi określenie REST, to może być ciężko (postaram się w niedalekiej przyszłości uzupełnić blog o wpisy wyjaśniające te zagdnienia). Teraz skupie się na totalnej praktyce i na tym jak zacząć przy *minimum* wiedzy i bez wysiłku (co nie oznacza, że jest to najlepszy sposób, powiem więcej, jest to sposób dla osób, które muszą coś zrobić na wczoraj albo wykonują bardzo proste skrypty dla SIEBIE).

## Uzyskanie dostępu do zasobów (uwierzytelnienie klienta)

### Shoper - panel administracyjny

Zacznijmy od stworzenia konta, z którego korzystać będzie nasz skrypt.

W menu wybieramy *Konfiguracja > Administracja, system*

<figure class="center">
	<img src='{{ site.url }}/images/shoper/shoper_menu_adminconf.gif' alt="">
	<figcaption>Konfiguracja > Administracja, system</figcaption>
</figure>

Możemy stworzyć nową grupę administratorów (np. nazywająć ją *admin*). Wybieramy w prawym górnym rogu opcje *Dodaj grupę administratorów*

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

Nadajemy odpowiednie uprawnienia grupie w której znajduje się nasze konto - proponuję nie przesadzać, lepiej jest dodawać uprawnienia w miare potrzeb, bo jak powiadają 
> "WITH GREAT POWER THERE MUST ALSO COME--GREAT RESPONSIBILITY!"

Klikamy na *nazwa grupy > Uprawnienia*
<figure class="center">
	<img src='{{ site.url }}/images/shoper/shoper_admingroup_prem.gif' alt="">
	<figcaption>Uprawnienia.</figcaption>
</figure>

#### Ok to na tyle co potrzebujemy z panelu administracyjnego, teraz czas na Python'a.

### Skrypt

Importujemy bibliotekę [Requests](http://docs.python-requests.org/en/latest/)

{% highlight python %}
import requests
{% endhighlight %}

Kolejnym krokiem będzie zainicjowanie sesji, która ułatwi nam robotę (będzie przechowywać parametry przekazywane razem z żądaniami, jak np. nagłówki, ciastka).

{% highlight python %}
s = requests.Session()
{% endhighlight %}

Jesteśmy gotowi do uzyskania token'a (w uproszczeniu jest to kod przekazywany w nagłówku, aby serwis REST mógł nas rozpoznać i udzielić dostępu do zasobów). Token uzyskujemy odwołując się do zasobu */auth* metodą POST, którą przekazujemy login i hasło do konta, stworzone wcześniej w panelu administracyjnym.

{% highlight python %}
response = s.post('https://<url_naszego_sklepu>/webapi/rest/auth', auth=(<login>, <haslo>))
{% endhighlight %}

Jeżeli login, hasło i adres naszego sklepu są prawidłowe, powinniśmy otrzymać odpowiedź do zmiennej *response*. Odpowiedzi zwykle są opisane w składni JSON, ale metoda post() zwraca nam obiekt. Wypisując do konsoli *response*, otrzymamy jedynie status odpowiedzi.

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

Uzyskaliśmy *access_token* i przypisaliśmy go do zmiennej *token*. Wypada teraz go użyć. Jak pisałem wcześniej, *access_token* musimy przekazywać jako element nagłówka każdego żądania. Utworzona sesja pomoże nam się z tym uporać. Sesja będzie dla nas przechowywać nagłówki i automatycznie "doklejać" je do każdego żądania, które zostaną wywołane wewnątrz niej. Dodajmy więc token do sesji.

{% highlight python %}
s.headers.update({'Authorization': 'Bearer %s' % token})
{% endhighlight %}

Od tej pory nasze żądania są uwierzytelnione, co oznacza, że możemy swobodnie korzystać z zasobów REST Api.

### Przykładowe odwołanie się do zasobu

Do dyspozycji, mamy operacje [CRUD](https://pl.wikipedia.org/wiki/CRUD). Wszystkie zasoby (opisane z przykładami) znajdziemy w [Dokumentacja REST API Shoper](https://developers.shoper.pl/developers/api/getting-started). Pozostaje jedynie dobrać odpowiednie zasoby i umiejętnie je wykorzystać. 

Na koniec mały przykład, jak uzyskać listę produktówa w naszym sklepie:

{% highlight python %}
response = s.get(URL + '/products')
print(response.json())
{% endhighlight %}

Warto zauważyć, że w odpowiedzi dostajemy tylko jedną ze stron, zawierającą określoną liczbę rekordów (kolekcje). Na tym zakończę ten post, ponieważ to, jak po kolei "przejść" po tych stronach (stronicowanie - czy jak coraz częściej się spotyka, paginacja), wymaga wiedzy jak dodawać do naszych rządań parametry, a o tym w następnej części o REST Api.




