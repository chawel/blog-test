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
Poprzednio zaprezentowałem działanie wtyczki Power Tools do Google SpreadSheet na konkretnym przykładzie: [Google SpreadSheets i Power Tools - Jak wydzielić produkty główne z wariantów?]()

Pojawiła się tam pewna konrtowersja, mianowicie, wtyczka wymagała dość wielu uprawnień (w tym pozwolenie na przetwarzanie danych na osobnym serwerze), oraz po okreśie próbnym należy wykupić coroczną subskrypcję (co nie jest tanie). Zrozumiałym jest więc, że dla niektórych osób są to czynniki dyskwalifikujące taką wtyczkę, z drugiej zaś strony oferuje ona na prawdę potężne możliwości. Czy trzeba jednak godzić się na takie warunki by ułatwić sobie pracę z arkuszami kalkulacyjnymi?

# Google Apps Script
Rozwiązaniem jest mechanizm pisania własnych skryptów dla dokumentów na platformie **Google Sheets**. Jeżeli znacie Excelowy VBA - język w którym wiele osób próbowało napisać coś pożytecznego, a następnie poddawało się w połowie - to powiem wam, jest to jego lepsza i prostsza wersja. 

[Google Apps Script](https://developers.google.com/apps-script/) oferuje nam prosty edytor skryptów, które piszemy w **języku opartym o JavaScript**, wykonując je i przechowując w chmurze. Możliwości są ogromne - jak proste formatowanie tekstu, automatyzacja czynności, ale też budowanie sieci zależności z innymi aplikacjami google, korzystanie z bibliotek JS, implementacje dla zaawansowanej obróbki danych i **jeszcze więcej.**

> Jeżeli chcesz w ogóle zacząć pisać skypty w *GAS*, musisz przynajmniej znać podstawy JavaScript. Nie trzeba być mistrzem, co pokazuje przykład mojej osoby, więc nie ma się czego bać.

**Google Apps Script** udostępnia możliwość pisania **samodzielnych skryptów**, które działać mogą na wielu usługach Google (np. autoresponder dla Gmaila z funkcją dodawania załączników do Google Drive, zbieranie informacji o aktywności z kalendarza do arkusza Google Sheet, itd.), **oraz takie które działają (zawierają się) w obrębie konkretnego pliku** (np. formatują tekst pliku Google Docs, obrabiają dane w akruszu, dodają nowe pozycje i funkcje w menu). 

> Warto pamiętać, że udostępniając dokumenty z zawartym skryptem, jego kod źródłowy będzie dostępny również dla osoby przeglądającej owy dokument - należy więc uważać, jeżeli chcemy udostępnić jedynie treść dokumentu (bez skryptu).

Skrypty samodzielne tworzymy przechodząc na stronę `script.google.com`, jednak im poświęcę czas innym razem. Teraz skupię się na tym drugim typie, konretnie przedstawię jak napisać prosty skrypt dla arkusza kalkulacyjnego. Oczywiście analogicznie możemy pisać skrypty dla *Google Docs* czy innych aplikacji z pakietu *Google Sheets*.

## Pierwszy skrypt (Google SpreadSheets)
Zaczynamy od otworzenia pliku Google SpreadSheets i stworzenia dla niego skryptu. 

Przejdź do **Tools > Script Editor** - powinien otworzyć się edytor.
To tutaj będziemy pisać nasz kod. Przykładowo, najprostsza funkcja wyświetlająca *alert* z napisem *"Cześć"*.

{% highlight javascript %}
function func() {
  Browser.msgBox("Cześć");
}
{% endhighlight %}

Ok, mamy funkcję, tylko jak ją teraz wywołać? Jest kilka sposobów, ale my skorzystamy w możliwości dodawania elementów do menu, aby można było wywołać ją z poziomu interfejsu *Google SpreadSheets*.

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

Możemy również skorzystać z możliwości uruchamiania funkcji z edytora, wystarczy że w menu wybierzemy funkcję `onLoad()` i klikniemy strzałkę.
> <GIF>

Dzięki opcji uruchamiania pojedynczych funkcji możemy sprawdzać działanie poszczególnych z nich. Wrócmy do naszego arkusza, powinna teraz pojawić się nowa pozycja w menu, którą możemy przetestować.
> <GIF>

## Praktyczne przykłady
Wiem, że przykład wyżej nie urywa dolnych części ciała, dlatego postaram się zrehabilitować i zaprezentować kilka skryptów, które są praktyczne i rzeczywiście mogą do czegoś się przydać

### Rozdzielanie wartości w komórkach
We wstępie wspomniałem o poprzednim wpisie, który pokazywał co można zdziałać z pewną wtyczką. Jedną z jej funkcji jest *Split* - która umożliwie rodzielanie wartości w danej kolumnie na osobne kolumny, po wskazanym znaku. Spróbujemy odtworzyć jej działanie.
{% highlight javascript %}
{% endhighlight %}

### Usuwanie polskich znaków
Nasz piękny język korzysta ze znaków diakretycznych, których czasami musimy się pozbyć.
{% highlight javascript %}
{% endhighlight %}

### Komentarz z datą edycji komórki
Pracując na jedym arkuszu z wieloma osobami, warto czasami wiedzieć czy przypadkiem coś się nie zmieniło (np. ktoś zakualizował ceny produktów).

{% highlight javascript %}
function onEdit(e){
  var sheet = e.source.getActiveSheet();
  var editedCell = sheet.getActiveCell();

  if(editedCell.getColumn() == 6){
    editedCell.setNote('Last modified: ' + new Date());
  }
}
{% endhighlight %}

### Szukanie duplikatów (procentowo)
W zasadzie ten przykład to taki *Proof of Concept*. Nie nadaje się on zbytnio do realnej pracy z danymi, chociaż przy bardzo małych arkuszach i niewielkiej długości tekstach można się nim pobawić (problemy z optymalizacją). Najistotniejszym elementem w tym przykładzie jest fakt, że możemy korzystać z zewnętnych bibliotek napisanych w JS!

Skrypt ma za zadanie odnajdowanie duplikatów, a raczej podobnych do siebie w określonym stopniu (wyrażonym procentowo) tekstów.

{% highlight javascript %}
{% endhighlight %}

# Podsumowanie
Po więcej informacji zapraszam do oficjalnej dokumentacji: [Dokumentacja Google Apps Script](https://developers.google.com/apps-script/guides/sheets)

Zapraszam wszystkich do komentowania, szczególnie zachęcam do opisywania swoich problemów - postaram się opisać ich rozwiązania w następnych wpisach. 
  





