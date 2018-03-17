---
layout: post
title: Pliki CSV i Python - odczyt i zapis
modified:
categories: python/tutorial
description:
tags: [csv, python, tutorial]
image:
  feature: csv_python.jpg
  credit:
  creditlink:
comments: true
share:
date: 2018-02-26T20:37:42+01:00
---

Niezwykle ważny temat, wręcz fundament dla każdego administratora sklepu internetowego, czyli pliki z danymi - tym razem na warsztat bieżemy te w formacie CSV. Omówię na szybko jak w prosty sposób zacząć z nich korzystać i ożenić je z naszymi skryptami w Pythonie. Skupie się na ich odczycie i zapisie danych.

<!-- more -->

## Ciut teorii i informacji o plikach CSV

Chociaż podejrzewam Was o znajomość z plikiem tego rodzaju, to jednak zaryzykuję i wytłumaczę któż on zaś.

> CSV (ang. comma-separated values, wartości rozdzielone przecinkiem) – format przechowywania danych w plikach tekstowych i odpowiadający mu typ MIME text/csv.

I to w zasadzie tyle z definicji o formacie CSV, bo w tym zdaniu zawarte są najistotniejsze informacje:
 - Są to pliki tekstowe (można bez problemu je otworzyć w przysłowiowym notatniku)
 - Wartości oddzielone są przecinkiem (lub innym separatorem np. tabulatorem, średnikiem)
 - Jest to format przechowywania danych w plikach typu text/csv (to znaczy, że jest to jakiś znany i uznawany format)
 - Dodam, że jest on chyba jednym z najprostszym i najbardziej rozpowszechnionym (na chwilę obecną) formatem (XML może być równie popularny, ale na pewno jest bardziej skomplikowany).

Przykładowy plik CSV może wyglądać tak (otwierając go w najprostszym edytorze tekstu):
```
kolumna 1,kolumna 2,kolumna 3
wartosc1_kol1,wartosc1_kol2,wartosc1_kol3
wartosc2_kol1,wartosc2_kol2,wartosc2_kol3
wartosc3_kol1,wartosc3_kol2,wartosc3_kol3
```

Ok, fajnie, tylko co nam to daje? A no skoro jest to szeroko rozpowszechniony format, możemy korzystać z jego konstrukcji w różnych aplikacjach, jak np. MS Excel czy [LibreOffice Calc](https://www.libreoffice.org/discover/calc/), które automatycznie sformatują go do postaci tabeli.

<img>

Na pewno wielką zaletą formatu CSV, jest uniwersalny charakter, możemy w nim przechowywać wszystko to, co może zostać zapisane w formie tekstu (znaki, ciągi znaków, liczby) i skorzysta na postaci w formie tabeli (czyli nazwa kolumny i wartość). Wiele aplikacji, takich jak sklepy internetowe, korzystają z tego formatu do wymiany informacji z użytkownikiem (bo jest przyjazny człowiekowi) ale również między innymi serwisami (np. hurtownia wystawia stany magazynowe klientom).

## Co na to Python?

Jak zwykle, mamy gotowy moduł w standardowej bibliotece służący do obsługi tego typu formatu. Jego nazwy *nikt* pewnie się nie domyślił, więc powiem że chodzi o [moduł csv](https://docs.python.org/3/library/csv.html). Standardowo zaczynamy od jego importu.
{% highlight python %}
import csv
{% endhighlight %}

### Odczyt

Skoro jest to plik tekstowy, to otwieramy go identycznie jak standardowy plik tekstowy (*thanks Captain Obvious!*).
{% highlight python %}
# konstrukcja *with* pozwala na otworzenie pliku i korzystanie z niego wewnątrz niej
# po jej opuszczeniu automatycznie zamknie strumień odczytu
with open('plik.csv', 'r') as csvfile:
    # deklarujemy nasz *czytacz*
    # parametr *delimiter* jest opcjonalny i wskazuje jaki został w pliku użyty separator
    csvreader = csv.reader(csvfile, delimiter=',')
{% endhighlight %}

#### Zatrzymajmy się tutaj na chwilę, ponieważ musimy omówić istotną sprawę - kodowanie.
Chodzi o to, aby przy odczycie z pliku polskich znaków ([litery diakrytyzowane](https://pl.wikipedia.org/wiki/Znaki_diakrytyczne)), nie zmieniały się one w tzw. *krzaki*. Temat kodowania, zwłaszcza w Pythonie, to temat rzeka, który zasługuje na nowy wpis (a nawet serię). Co ważne, musicie pamiętać, że wszystkie wpisy na blogu traktują o najnowszej wersji Pythona (3.5+). W skrócie, wersja Python 2 w inny sposób traktuje typ `string` a co za tym idzie i jego kodowanie. 

Kolejnym istotnym pojęciem jest **Unicode (UTF-8)**, znowu w skrócie, jest to uniwersalny zestaw znaków (jakby zbiór) w komputerach, stworzony by zawierał wszystkie znaki ze wszystkich języków. Tyle teorii. W praktyce, obecnie większość plików tekstowych zapisujemy właśnie w kodowaniu `UTF-8`, nie bawimy się już w `ISO 8859-2` (mam nadzieję). Daje to możliwość umieszczania w takim tekcie wszelkich liter diakrytyzowanych, z różnych języków, i nikt nie musi się głowić *"hmm.. krzaki... jakie tu jest kodowanie?"*. 

#### Dlatego dla uproszczenia: Zapisujmy pliki w kodowaniu UTF-8, i otwierajmy je też w UTF-8.

Otwierając plik, chcemy wskazać jego kodowanie (nie musi to być w każdym przypadku `utf-8`, ktoś mógł użyc innego kodowania do zapisu tego pliku, ale jeżeli są *krzaki* to najłatwiej jest spróbować otworzyć z `utf-8`).
{% highlight python %}
# pojawia się nowy paramet encoding ustawiony na utf-8
with open('plik.csv', 'r', encoding='utf-8') as csvfile:
    csvreader = csv.reader(csvfile, delimiter=',')
{% endhighlight %}

Kontynuując, pamiętajmy jak wygląda format pliku CSV, jest on nie tylko podzielony na kolumny ale również na wiersze, to też będziemy *czytać* po kolei, następujące po sobie linie.
{% highlight python %}
    for row in csvreader:
        print(row[0]) # wartość kolumny 1 z tego wiersza
        print(row[1]) # analogicznie - 2 kolumna
        print(row[2]) # 3cia
{% endhighlight %}

#### output:
```
kolumna 1
kolumna 2
kolumna 3
wartosc1_kol1
wartosc1_kol2
wartosc1_kol3
wartosc2_kol1
wartosc2_kol2
wartosc2_kol3
wartosc3_kol1
wartosc3_kol2
wartosc3_kol3
```

Dlaczego odwołujemy się do `row[n]`? Każdy wiersz zczytywany jest do listy, na przykład:
```
W pliku:
wartosc1_kol1,wartosc1_kol2,wartosc1_kol3

W Pythonie:
[wartosc1_kol1, wartosc1_kol2, wartosc1_kol3]
```

Jak widać możemy odwoływać się do indeksów wartości tej listy. Co jeżeli nie znamy numeru kolumny w której występuje interesująca nas wartość, a jedynie jej nazwę? 

Kolumny mają swoje nazwy zdefiniowane w pierwszym wierszu pliku CSV. Wystarczy, że zamiast do zwykłej listy (tablica 2D), wczytamy kolejne wiersze jako słownik ([dict](https://docs.python.org/3/library/stdtypes.html#dict)) - co da nam strukturę w stylu
```
{nazwa_kolumny: wartość, ..., nazwa_kolumny_n: wartość_n}
```

W tym celu musimy użyć bardziej wyrafinowanego *czytacza*.
{% highlight python %}
with open('plik.csv', 'r', encoding='utf-8') as csvfile:
    csvreader = csv.DictReader(csvfile)

    for row in csvreader:
        print(row['kolumna 1'])
        print(row['kolumna 2'])
        print(row['kolumna 3'])
{% endhighlight %}

#### output:
```
wartosc1_kol1
wartosc1_kol2
wartosc1_kol3
wartosc2_kol1
wartosc2_kol2
wartosc2_kol3
wartosc3_kol1
wartosc3_kol2
wartosc3_kol3
```

### Zapis

Łatwo się domyślić, że to co tutaj omówię, będzie po prostu odwróceniem procesu odczytu. Żeby nie przeciągać od razu zacznę od kodu i go omówię.
{% highlight python %}
# zwroc uwage na zmiane symbolu trybu otwarcia pliku 'r' na 'w'
# 'r' to read czyli odczyt, 'w' oznacza write czyli zapis
# istnieją inne, ale nie będą one na tym etapie potrzebne
with open('plik.csv', 'w', encoding='utf-8') as csvfile:
    # inicjujemy *zapisywacz*
    csvwriter = csv.writer(csvfile)
    # wpisujemy pierwsza linie naszego pliku CSV (nazwy kolumn)
    csvwriter.writerow('kolumna 1', 'kolumna 2', 'kolumna 3')
    # czas na kolejne linie z wartosciami
    csvwriter.writerow('wartosc1_kol1', 'wartosc1_kol2', 'wartosc1_kol3')
    csvwriter.writerow('wartosc2_kol1', 'wartosc2_kol2', 'wartosc2_kol3')
    csvwriter.writerow('wartosc3_kol1', 'wartosc3_kol2', 'wartosc3_kol3')
{% endhighlight %}


Metoda `writerow()` zapisuje linię do pliku. To znaczy, że możemy ją *zapętlić* i zapisywać nieokreśloną ilość danych. Trywialny przykład (i również bezsensowny), czyli jak zapisać tabliczkę mnożenia (rozmiar 10x10):
{% highlight python %}
with open('plik.csv', 'w', encoding='utf-8') as csvfile:
    csvwriter = csv.writer(csvfile)
    # metoda range() to generator listy liczb (w uproszczeniu), pamietaj ze przedział podany to <od, do)
    # czyli n nie bedzie rowne w tym przypadku 11, ale tylko mniejsze
    for n in range(1, 11):
        csvwriter.writerow([n*1, n*2, n*3, n*4, n*5, n*6, n*7, n*8, n*9, n*10])
{% endhighlight %}

A teraz troche bardziej praktyczny przykład, czyli zapis struktur do pliku CSV (typu `dict`)
{% highlight python %}
struct = {'kolumna_1': 'wartosc',
          'kolumna_2': 'wartosc',
          'kolumna_3': 'wartosc'}

struct2 = {'kolumna_2': 'wartosc',
           'kolumna_3': 'wartosc',
           'kolumna_4': 'wartosc'}

struct3 = {'kolumna_1': 'wartosc',
           'kolumna_3': 'wartosc',
           'kolumna_4': 'wartosc'}

struct_list = [struct, struct2, struct3]

with open('plik.csv', 'w') as csvfile:
    # definiujemy nagłówek (czyli nasze kolumny)
    fieldnames = ['kolumna_1', 'kolumna_2', 'kolumna_3', 'kolumna_4']
    csvwriter = csv.DictWriter(csvfile, fieldnames=fieldnames)

    # zapisujemy do pierwszej linii zdefiniowane wczesniej nazwy kolumn
    csvwriter.writeheader()

    # zapisujemy do pliku po kolei nasze struktury iterujac po ich liscie
    for n in struct_list:
        csvwriter.writerow(n)
{% endhighlight %}

Zauważ, że nie musimy wypełniać wartości dla wszystkich kolumn w danym wierszu, ale musimy zdefiniować wszystkie używane kolumny (*header*).

CSV to oczywiście nie jedyny format danych, warto przy okazji wspomnieć tutaj o XML, JSON, YAML i innych, ale to materiał na następne wpisy, które na pewno się pojawią na moim blogu. Zachęcam więc do śledzenia wpisów.


