---
layout: post
title: Allegro.pl - Automatyczne grupowanie aukcji (wielowariantowość)
modified: Jak automatycznie połączyć kilka aukcji (wariantów) w jedną
categories: ecommerce/allegro
description:
tags: [allegro, python, selenium]
image:
  feature: captcha_robot_small.gif
  credit: Captcha robot
  creditlink: https://www.youtube.com/watch?v=fsF7enQY8uI
comments:
share:
date: 2018-02-14T21:48:21+01:00
---

Allegro wprowadziło jakiś już czas temu, możliwość *grupowania* naszych aukcji po konkretnym dla kategorii i wspólnym dla każdej z nich parametrze (a nawet parze parametrów). Służy do tego narzędzie w panelu naszego konta na Allegro.pl. Wizja ręcznego grupowania tysiąca aukcji, które wystawiliśmy automatycznie, może spędzać sen z powiek, dlatego zaprezentuję jedną z opcji w jaki sposób ułatwić sobie to zadanie.

<!-- more -->

Zacznijmy od tego, że Allegro nie udostępniło jeszcze żadnego zasobu w API, którym moglibyśmy łatwo łączyć aukcje ze sobą. Pytania o termin wprowadzenia jakiegoś rozwiązania, umożliwiającego choćby częściowe zautomatyzowanie tworzenia ofert wielowariantowych, zadawane są nieustannie od prawie roku. Odpowiedź jest zwykle ta sama: "Za kilka miesięcy" lub inny bliżej [nieokreślony](http://idn.org.pl/sonnszz/nieokreslony.htm) termin.

> Jak żyć?

Spokojnie, pokażę pewien sposób, który imitować będzie działania użytkownika. Nie jest to ani eleganckie, ani niezawodne rozwiązanie, ale miejmy nadzieję że wystarczy do czasu wprowadzenia przez Allegro zasobu do REST API. 

Skorzystam z narzędzia Selenium w połączeniu z Python'em i szczyptą biblioteki Requests.

## Rekonesans (czyli jak to ugryźć)

Jak już pewnie wspominałem, automatyzacja polega na bezobsługowym (samoczynnym) wykonywaniu się zadań (najczęściej serii pojedynczych czynności), które musimy sobie zdefiniować. Nie ma tu magii, ten skrypt będzie "udawać", że za każdą z tych czynności stoi człowiek (a przynajmniej *coś* co otworzyło przeglądarkę). Skoro tak, to musimy rozeznać się co konretnie dzieje się w czasie łączenia ofert. Nie będziemy tworzyć sztucznej inteligencji, ani też robota który operować będzie kursorem i kliknięciami. Mam w zanadrzu inną metodę. Niezbętnym narzędziem na tym etapie, będzie sławne F12 (w większości nowoczesnych przeglądarek jest to skrót otwierający tzw. *Narzędzia programistyczne*). Przedstawiam proces rozkminy.

Dla procesu logowania potrzebujemy poznać budowę formualrza.
<figure class="center">
	<img src='{{ site.url }}/images/allegro/allegro_form.gif' alt="">
	<figcaption>Uzyskanie nazw pól formularza logowania.</figcaption>
</figure>

Teraz, podglądamy co dzieje się za kulisami, kiedy łączymy ze sobą aukcje. Wchodzimy do zakładki *Moja Sprzedaż > Wielowariantowość* a następnie:
{% capture images %}
	/images/allegro/allegro_variants_editor1.jpg
	/images/allegro/allegro_variants_editor2.jpg
	/images/allegro/allegro_variants_editor3.jpg
	/images/allegro/allegro_variants_editor4.jpg
{% endcapture %}
{% include gallery images=images caption="Podglądanie allegro.pl" cols=2 %}

W przypadku pierwszego kroku, czyli po wybraniu aukcji z jednej kategorii, uzyskujemy informacje na temat wszystkich możliwych do wykorzystania parametrów do łączenia ich ze sobą - w tym przypadku z kategorii *Chodniki*
{% highlight json %}
...
{"attributes":[{"id":"2978","name":"Szerokość"},{"id":"11323","name":"Stan"},{"id":"11509","name":"Kolor"},{"id":"11525","name":"Materiał wykonania"},{"id":"17448","name":"Waga (z opakowaniem)"}]
...
{% endhighlight %}

Po zatwierdzeniu, widzimy jakie parametry przekazujemy w żądaniu. Możemy je z łatwością rozpoznać i wykorzystać - tutaj łączenie po atrybucie *Szerkość* bez łączenia po "zdjęciu"
{% highlight json %}
{"name":"test","offers":[{"offerId":"592...099"},{"offerId":"592...080"}],"attributes":["2978"],"imageAttribute":false}
{% endhighlight %}

## Logowanie

Ok, wszystko ładnie, tylko skoro są żądania, jest zasób do którego je wysyłamy, są odpowiedzi w formie JSON - to na co nam te całe "udawanie" i Selenium? Szczerze - z lenistwa, bo prawdopodobnie jest bardziej wyrafinowany sposób jak obejść cały proces logowania bez otwierania przeglądarki, ale byłoby to bardzo trudne i czasochłonne (a po to właśnie automatyzujemy - aby oszczędzić czas i ~~oddawać się błogiemu nieróbstwu~~ poświęcać się rzeczom ważniejszym).

Dla zainteresowanych, powiem tylko, że Allegro.pl stosuje zabezpieczenia przed ~~XSS~~ [CSRF](https://pl.wikipedia.org/wiki/Cross-site_request_forgery), wiele elementów w plikach cookies, a co najgorsze... Niekiedy przy logowaniu może pojawić się wymóg rozwiązania **CAPTCHA**.

> A wild CAPTCHA appeard!

A Selenium jest niczym pizza dla zgłodniałego informatyka. Nie trzeba się wysilać, bo otwierana przezeń przeglądarka (driver) zachowa wszelkie pozory, jak i wymagane pliki cookie, headery, oraz daje graficzny interfejs w razie CAPTCH'y.

### Implementacja procesu automatycznego logowania (prawie*)

*\*Prawie - bo w razie CAPTCHA, musimy wysilić się na kliknięcie w "Nie jestem robotem". Nie martw się, to będzie jedyny Twój wkład w działanie skryptu (a jest duże prawdopodobieństwo, że nigdy się to nie wydarzy - ja doświadczyłem tej ciężkiej pracy ~~aż~~ tylko raz na kilkanaście wywołań tego skryptu).~*

Zaczynając, zaimportujmy potrzebne biblioteki
* [Selenium](https://selenium-python.readthedocs.io/installation.html)

# STOP!
### Jednak użyjemy biblioteki łączącej w sobie 2 biblioteki: Selenium i Requests (da nam to możliwość wysyłania reqestów bezpośrednio z przeglądarki)
* [Selenium-Requests](https://github.com/cryzed/Selenium-Requests) z driverem od [Firefox](https://github.com/mozilla/geckodriver/releases) (czyli imitować będzie przeglądarkę z liskiem)
{% highlight python %}
import seleniumrequests import Firefox
{% endhighlight %}

#### Zdażają się problemy z korzystaniem ze ściągniętego WebDriver'a, dlatego obszerna instrukcja instalacji:
> Instalacja dla Windows: Ściągamy odpowiednią paczkę z [repozyturum Mozilli](https://github.com/mozilla/geckodriver/releases) i rozpokowujemy. Aby użyć webdriver w skrypcie podamy pełną ściężkę do tego pliku (albo jak kto woli, można dodać ścieżkę do pliku .exe do PATH i korzystać)

> Instalacja dla Linux: Ściągamy odpowiednią paczkę z [repozyturum Mozilli](https://github.com/mozilla/geckodriver/releases) i rozpokowujemy. Plik umieszczamy w katalogu /usr/bin lub /usr/local/bin, lub dodajemy do PATH)

Dodajmy jeszcze 2 biblioteki
{% highlight python %}
import time
import json
{% endhighlight %}

Inicjujemy nasz WebDriver
{% highlight python %}
webdriver = Firefox()
{% endhighlight %}

##### Jeżeli pracujesz na systemie Windows, i nie dodałeś ścieżki do ściągniętego *geckodriver* do PATH, to możesz po prostu w deklaracji przekazać jako parametr pełną ścieżkę do niego
{% highlight python %}
webdriver = Firefox(r'C:\path\to\webdriver\geckodriver.exe')
{% endhighlight %}

Idzie dobrze, więc od razu otwieramy naszą przeglądarkę
{% highlight python %}
webdriver.get('https://allegro.pl/myaccount/')
{% endhighlight %}

Aby wypełnić automatycznie formularz logowania
{% highlight python %}
login_box = webdriver.find_element_by_name('username')
login_box.send_keys('twojlogin')

passwd_box = webdriver.find_element_by_name('password')
passwd_box.send_keys('twojehaslo')
{% endhighlight %}

Wysyłamy nasze dane, czyli potwierdzamy formularz (tutaj damy chwile oddechu dla skryptu - na wypadek CAPTCHY musimy miec czas na reakcje)
{% highlight python %}
login_box.submit()
# daj czas na wypadek captcha
# 10 sekund
time.sleep(10)
{% endhighlight %}

Ok, w tym momencie naszym oczom ukaże się widok panelu konta na allegro. To znaczy, że każdy *request* który puścimy będzie już uwierzytelniony. Przygotujmy więc paczkę do wysłania.

{% highlight python %}
offer1 = {'offerId': <numer_aukcji1>}
offer2 = {'offerId': <numer_aukcji2>}

offer_list = [offer1, offer2]

params = dict()
params['name'] = <nazwa_grupy>
params['attributes'] = [<id_atrybutu1>, <opcjonalnie_id_atrybutu2>]
params['imageAttribute'] = False # True - kiedy laczymy po "zdjeciach"
params['offers'] = offer_list # lista naszych ofert do polaczenia

headers = {"Content-Type": "application/vnd.allegro.public.v2+json"}
{% endhighlight %}

Myślę, że kod jest wystarczająco czytelny i prosty do zrozumienia. Elementy oznaczone klamrami '<' i '>' trzeba zastąpić odpowiednimi infomacjami. Co może wzbudzić ciekawość niektórych gałganów, to ten dziwny *header*. Cóż można powiedzieć, tak musi być i tyle - więc radze nie zapomnieć dodać go do żądania.

Pozostaje nam jedynie grzecznie zapytać allegro czy uprzejmie połączy nasze aukcje i dowiedzieć się co on na to, czyli odebrać odpowiedź i przekonwertować ją na czytelny dla nas format.

> "Would you kindly... Powerful phrase. Familiar phrase?" - Andrew Ryan

{% highlight python %}
response = webdriver.request('POST', 'https://allegro.pl/variants/', data=json.dumps(params), headers=headers)

print(response.json())
{% endhighlight %}

## W praktyce

Najdogodniejszą sytuacją jest, kiedy mamy dostęp do bazy SQL lub chociaż pliku CSV z numerami aukcji przypisanymi do jakiegoś identyfikatora wspólnego dla danych wariantów (parent_id, lub przedrostek w kodzie produktu, cokolwiek co może pomóc w automatycznym zgrupowaniu numerów aukcji). Wtedy wystarczy zapętlić nasz kod u góry i cieszyć się z automatycznego grupowania ofert.

Jeżeli nie zrobiliśmy pliku, nie mamy bazy z danymi, możemy próbować łączyć aukcje po np. tytule aukcji, który jeżeli jest w jakiejś części taki sam dla wszystkich wariantów - to można coś ugrać, ale o tym w kolejnych wpisach na blogu.

## BONUS

Jeżeli chcemy uzyskać w formie JSON oferty dowolnego sprzedawcy, możemy odwołać się do zasobu:
`https://allegro.pl/variants-editor/offers/search?sellerId=<id_sprzedawcy>`

*id_sprzedawcy* znajdziemy np. na aukcji, w odnośniku do jego ocen (wystarczy kliknąć na nick i w miejscu *id_sprzedawcy* będzie numer - o ten właśnie chodzi.
`https://allegro.pl/uzytkownik/<id_sprzedawcy>/oceny`

Dodatkowo, można odfiltrowywać konkretne numery aukcji:
`https://allegro.pl/variants-editor/offers/search?sellerId=<id_sprzedawcy>&query=<numer_aukcji>`

