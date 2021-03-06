---
layout: post
title: Google Apps Script - wprowadzenie
modified:
categories: inne
description: Wprowadzenie do Google Apps Script, czyli skrypty dla dokumentów Google.
tags: [excel, spreadsheets, google apps script]
image:
  feature: gsheet_script.png
  credit: Google Apps Script and SpreadSheets
  creditlink:
comments:
share:
date: 2018-05-31T17:19:54+02:00
---
Kontynuacja serii o obróbce danych. Jak poradzić sobie w sytuacji, kiedy żadna wtyczka czy program nie posiada niezbędnych dla nas funkcjonalności? Google Apps Script pozwala na pisanie własnych rozwiązań dla usług od Google.

<!-- more -->

# Wstęp
Poprzednio zaprezentowałem działanie wtyczki *Power Tools* do *Google SpreadSheet* na konkretnym przykładzie: [Google SpreadSheets i Power Tools - Jak wydzielić produkty główne z wariantów?]({{ site.baseurl }}{% post_url /inne/2018-05-13-google-spreadsheets-i-power-tools-cz-1 %})

Pojawiła się tam pewna kontrowersja, mianowicie, wtyczka wymagała dość wielu uprawnień (w tym pozwolenie na przetwarzanie danych na osobnym serwerze), oraz po okresie próbnym należy wykupić subskrypcję (co nie jest tanie). Zrozumiałym jest więc, że dla niektórych osób są to czynniki dyskwalifikujące taką wtyczkę, z drugiej zaś strony oferuje ona na prawdę potężne możliwości. Czy trzeba jednak godzić się na takie warunki by ułatwić sobie pracę z arkuszami kalkulacyjnymi?

# Google Apps Script
Rozwiązaniem jest mechanizm pisania własnych skryptów dla dokumentów na platformie **Google Sheets**. Jeżeli znacie Excelowy VBA - język w którym wiele osób próbowało napisać coś pożytecznego, a następnie poddawało się w połowie - to powiem wam, jest to jego lepsza i prostsza wersja. 

[Google Apps Script](https://developers.google.com/apps-script/) oferuje nam prosty edytor skryptów, które piszemy w **języku opartym o JavaScript**, wykonując je i przechowując w chmurze. Możliwości są ogromne - jak proste formatowanie tekstu, automatyzacja czynności, ale też budowanie sieci zależności z innymi aplikacjami google, korzystanie z bibliotek JS, implementacje dla zaawansowanej obróbki danych i **jeszcze więcej.**

> Jeżeli chcesz w ogóle zacząć pisać skrypty w *GAS*, musisz przynajmniej znać podstawy JavaScript. Nie trzeba być mistrzem, co pokazuje przykład mojej osoby, więc nie ma się czego bać.

**Google Apps Script** udostępnia możliwość pisania **samodzielnych skryptów**, które działać mogą na wielu usługach Google (np. autoresponder dla Gmaila z funkcją dodawania załączników do Google Drive, zbieranie informacji o aktywności z kalendarza do arkusza Google Sheet, itd.), **oraz takie które działają (zawierają się) w obrębie konkretnego pliku** (np. formatują tekst pliku Google Docs, obrabiają dane w arkuszu, dodają nowe pozycje i funkcje w menu). 

> Warto pamiętać, że udostępniając dokumenty z zawartym skryptem, jego kod źródłowy będzie dostępny również dla osoby przeglądającej owy dokument - należy więc uważać, jeżeli chcemy udostępnić jedynie treść dokumentu (bez skryptu).

Skrypty samodzielne tworzymy przechodząc na stronę `script.google.com`, jednak im poświęcę czas innym razem. Teraz skupię się na tym drugim typie, konkretnie przedstawię jak napisać prosty skrypt dla arkusza kalkulacyjnego. Oczywiście analogicznie możemy pisać skrypty dla *Google Docs* czy innych aplikacji z pakietu *Google Sheets*.

## Pierwszy skrypt (Google SpreadSheets)
Zaczynamy od otworzenia pliku [Google SpreadSheets](https://docs.google.com/spreadsheets/u/0/) i stworzenia dla niego skryptu. 

Przejdź do **Tools > Script Editor** - powinien otworzyć się edytor.
To tutaj będziemy pisać nasz kod. Przykładowo, najprostsza funkcja wyświetlająca *alert* z napisem *"Cześć"*.

{% highlight javascript %}
function func() {
  Browser.msgBox("Cześć");
}
{% endhighlight %}

Ok, mamy funkcję, tylko jak ją teraz wywołać? Jest kilka sposobów - my skorzystamy z możliwości dodawania elementów do menu, aby można było wywołać ją z poziomu interfejsu *Google SpreadSheets*.

{% highlight javascript %}
// jest to funkcja która uruchamia się wraz z otwarciem pliku
// należy do tzw. triggerów, które reagują na pewne działania
// czyli wykonują się w konkretnej sytuacji
function onOpen() {
  // Odwołujemy się do naszego arkusza i jego metody
  // dla pobrania całego UI
  SpreadsheetApp.getUi()
      .createMenu('Nasz dodatek') // Tworzymy nową pozycję w głównym menu
      .addItem('Nasza funkcja', 'func') // Dodajemy opcję która uruchomi wskazaną w drugim parametrze funkcję 
      .addToUi();
}

function func() {
  Browser.msgBox("Cześć");
}
{% endhighlight %}

W tym momencie za każdym razem gdy otworzymy nasz dokument, zostanie dodane do menu element **Nasz dodatek** który posiadać będzie opcję **Nasza funkcja** po wybraniu której uruchomi się funkcja `func()` naszego skryptu. Pamiętajmy o tym, że *trigger* nastawiony jest na otwarcie pliku, więc jeżeli nie odświeżymy / otworzymy ponownie naszego dokumentu, to nie będziemy widzieć nowej pozycji w menu. 

> Listę *triggerów* można znaleźć w dokumentacji: [Simple Triggers - Google Apps Script](https://developers.google.com/apps-script/guides/triggers/)

Możemy również skorzystać z możliwości uruchamiania funkcji z edytora, wystarczy że w menu wybierzemy funkcję `onOpen()` i klikniemy strzałkę.
<figure class="center">
	<img src='{{ site.url }}/images/gas/run_script.gif' alt="">
	<figcaption>Uruchamianie funkcji</figcaption>
</figure>

Dzięki opcji uruchamiania pojedynczych funkcji możemy sprawdzać działanie poszczególnych z nich. Wróćmy do naszego arkusza, powinna teraz pojawić się nowa pozycja w menu, którą możemy przetestować.
<figure class="center">
	<img src='{{ site.url }}/images/gas/hello_world_script.gif' alt="">
	<figcaption>Hello world</figcaption>
</figure>

### Istotna uwaga! (Bezpieczeństwo)
Google pilnuje nas, abyśmy nie uruchamiali skryptów, które nie zostały zweryfikowane. Nasz skrypt też nie został zweryfikowany, co za tym idzie, wyświetli się nam ostrzeżenie - które musimy ominąć jeżeli wiemy co robimy i co uruchamiamy (jeżeli to nasz skrypt i wiemy co tam jest, to wyrażamy zgodę na odpalenie).
<figure class="center">
	<img src='{{ site.url }}/images/gas/warning_script.gif' alt="">
	<figcaption>Czujne Google</figcaption>
</figure>

# Praktyczne przykłady
Wiem, że przykład wyżej nie urywa dolnych części ciała, dlatego postaram się zrehabilitować i zaprezentować kilka skryptów, które są praktyczne i rzeczywiście mogą do czegoś się przydać

## Komentarz z datą edycji komórki
Pracując na jednym arkuszu z wieloma osobami, warto czasami wiedzieć czy przypadkiem coś się nie zmieniło (np. ktoś zaktualizował ceny produktów).
[Kod na Github](https://gist.github.com/chawel/7381073687a254938355b5c50d5e4c62)


{% gist 7381073687a254938355b5c50d5e4c62 %}

## Wydzielanie wierszy z unikalnym wystąpieniem wartości w danym zakresie
Skrypt tworzy nowy arkusz i kopiuje do niego nagłówek (pierwszą linię arkusza na którym działamy). Następnie wyszukuje w zaznaczonym zakresie (zaznaczamy tylko jedną kolumnę!) pierwsze wystąpienia wartości (to znaczy, że duplikaty nie zostaną skopiowane) i kopiuje je do tego nowo stworzonego arkusza. Jest to moje rozwiązanie problemu opisanego w poscie o wydzielaniu produktów głównych. 
[Kod na Github](https://gist.github.com/chawel/c70d36ad83e09cb003a72103dec90292)

{% gist c70d36ad83e09cb003a72103dec90292 %}

## Szukanie duplikatów (procentowo)
W zasadzie ten przykład to taki *Proof of Concept*. Nie nadaje się on zbytnio do realnej pracy z danymi, chociaż przy bardzo małych arkuszach i niewielkiej długości tekstach można się nim pobawić (problemy z optymalizacją). Najistotniejszym elementem w tym przykładzie jest fakt, że możemy korzystać z zewnętrznych bibliotek napisanych w JS!

Skrypt ma za zadanie odnajdowanie duplikatów, a raczej podobnych do siebie w określonym stopniu (wyrażonym procentowo) tekstów.

Wystarczy wkleić zawartość biblioteki [difflib-browser](https://raw.githubusercontent.com/qiao/difflib.js/master/dist/difflib-browser.js) do projektu skryptu jako plik `difflib.gs` a właściwy skrypt w drugim pliku np. `find_uniques.gs`. [Kod na Github](https://gist.github.com/chawel/7e4e5b11544ab5a5ccb13e0a333d23d6)

{% gist 7e4e5b11544ab5a5ccb13e0a333d23d6 %}

# Podsumowanie
Po więcej informacji zapraszam do oficjalnej dokumentacji: [Dokumentacja Google Apps Script](https://developers.google.com/apps-script/guides/sheets)

Zapraszam wszystkich do komentowania, szczególnie zachęcam do opisywania swoich problemów - postaram się opisać ich rozwiązania w następnych wpisach. 
  





