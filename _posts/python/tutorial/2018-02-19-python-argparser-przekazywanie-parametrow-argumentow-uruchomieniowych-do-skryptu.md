---
layout: post
title: Python argparse - przekazywanie parametrów (argumentów) wiersza poleceń
modified:
categories: python/tutorial
description: Poradnik dla początkujących, jak wykorzystać moduł argparse do obsługi argumentów w skryptach Python.
tags: [python, poradnik, argparse]
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2018-02-19T19:38:37+01:00
---

Mamy skrypt który wykonuje jakąś operację na konkretnym pliku - czy nie lepiej aby mógł on wykonywać operacje na dowolnych plikach? No tak, ale komu się chce co raz edytować kod źródłowy, aby podmienić nazwę tego pliku... Dobrze, że istnieją argumenty uruchomieniowe! Dzięki nim bez problemu możemy przekazywać wartości (parametry) z *zewnątrz* - najczęściej w trakcie uruchamiania z konsoli (wiersza poleceń). Postaram się pokazać ich praktyczne zastosowania oraz jak je "parsować" (*ang. parse*).

<!-- more -->

## Wstęp

Jeżeli kiedykolwiek korzystałeś z Linux'a to pewnie nie raz spotkałeś się z dodatkowymi elementami podawanym zaraz po wywoływanej komendzie. Dla tych, który jeszcze nie bardzo kojarzą, spróbuję przybliżyć ten koncept.

Standardowe polecenie do listowania plików. Po zatwierdzeniu, w konsoli zostanie wypisana lista plików bieżącego katalogu (ten w którym się znajdujemy w momencie wywołania komendy).
```
$ ls
plik.txt plik2.txt plik3.txt
```

Ok, ale co jeżeli chcemy sprawdzić rozmiary tych plików? W świecie bez argumentów wiersza poleceń, musielibyśmy skorzystać z innej komendy. Na szczęście tak nie jest.
```
$ ls -s
total 15664
2048 plik1.txt  7472 plik2.txt  6144 plik3.txt
```

Liczby obok nazw plików to wartości w kilobajtach. Co jeżeli interesuje nas ile to jest MB? Wyjmujemy kalkulator? Można, ale łatwiej będzie dodać kolejny argument do naszej komendy
```
$ ls -sh
total 16M
2,0M plik1.txt  7,3M plik2.txt  6,0M plik3.txt
```

I tak dalej, i tak dalej. Jak widać, możemy nasze argumenty łączyć ze sobą, podając je w jednym ciągu. Wyróżniamy argumenty obowiązkowe, bez których komenda się nie wykona, oraz te opcjonalne (jak widać wyżej). 

Ostatni przykład, komenda *touch* (służy do tworzenia "pustych" plików), jej argument obowiązkowy to nazwa pliku wyjściowego, gdy go nie podamy, komenda się nie wykona i wyświetli nam błąd.
```
$ touch
touch: missing file operand
Try 'touch --help' for more information.
```

Więc aby stworzyć plik o nazwie *plik4.txt* podajemy go bezpośrednio po komendzie (w jednej linii).
```
$ touch plik4.txt

$ ls
plik1.txt  plik2.txt  plik3.txt  plik4.txt
```

Oczywiście, jest to bardzo pobieżne wytłumaczenie i nie wyczerpuje tematu. Liczę na to, że tym co na początku nie mieli zielonego pojęcia, pozwoli zrozumieć dalszą część tego wpisu.

## Moduł argparse

Python w swojej standardowej bibliotece posiada bardzo sprytny i ułatwiający obsługę argumentów linii komend moduł o nazwie [argparse](https://docs.python.org/3.5/library/argparse.html) którego teraz będziemy używać. Warto wspomnieć, że jest to najłatwiejszy sposób, ale nie jedyny. Na upartego możemy "zejść poziom niżej" i zrobić to samo przy użyciu [sys.argv](https://docs.python.org/3/library/sys.html#sys.argv), ale uważam że nawet dla prostych skryptów, lepiej jest używać *argparse*.

### Przykład

Bez zbędnych formalności, zacznijmy od przykładu użycia w skrypcie.

Importujemy moduł
{% highlight python %}
import argparse
{% endhighlight %}

Inicjujemy *parser* (to on odpowiada za poprawne "wychwycenie" podawanych argumentów i odczytanie ich wartości)
{% highlight python %}
parser = argparse.ArgumentParser(description='Tutaj możemy podać zwięzły opis naszego skryptu')
{% endhighlight %}

Definiujemy możliwe do użycia argumenty.
{% highlight python %}
parser.add_argument('filename', help="Opis tego argumentu (krótkie wyjaśnienie co przyjmuje) - tutaj uznajmy że nazwę pliku")
{% endhighlight %}

Parsujemy argumenty, aby "wyłuskać" przekazane w nich wartości i wyświetlamy je na ekranie
{% highlight python %}
args = parser.parse_args()
print("Podana nazwa pliku: ", args.filename)
{% endhighlight %}

Spróbujmy teraz wykonać nasz skrypt (zakładając że nazwaliśmy go *py.py*)
```
$ python py.py
usage: py.py [-h] filename
py.py: error: the following arguments are required: filename
```

Jak widać, argument *filename* jest obowiązkowy, więc bez niego nasz skrypt nie ruszy
```
$ python py.py nazwa_pliku
Podana nazwa pliku:  nazwa_pliku
```

Dodajmy kolejny argument, tym razem opcjonalny
{% highlight python %}
parser.add_argument('-a', '--argument', help="Nasz opcjonalny argument", required=False)
{% endhighlight %}


Jeżeli chcemy, aby nikt się nie pomylił i w np. argumencie który ma przyjmować ilość (*integer*) nie spisał wartości innego typu (np. *string*), możemy go zdefiniować. Co więcej, możemy zadeklarować domyślną wartość argumentu, która zostanie do niego przypisana w przypadku gdy użytkownik jej nie zdefiniuje (pominie ten argument).
{% highlight python %}
parser.add_argument('-i', '--ilosc', help="Argument o konkretnym typie i wartością domyślną", type=int, default=5, required=False)
{% endhighlight %}

Zobaczmy jak nasz skrypt radzi sobie z argumentami
```
$ python py.py nazwa_pliku
Podana nazwa pliku:  nazwa_pliku
Wartosc argumentu dodatkowego:  None
Wartosc argumentu z typem i domyslna wartoscia:  5

$ python py.py nazwa_pliku -a test -i 1000
Podana nazwa pliku:  nazwa_pliku
Wartosc argumentu dodatkowego:  test
Wartosc argumentu z typem i domyslna wartoscia:  1000

$ python py.py nazwa_pliku --argument test_long --ilosc 10000
Podana nazwa pliku:  nazwa_pliku
Wartosc argumentu dodatkowego:  test_long
Wartosc argumentu z typem i domyslna wartoscia:  10000

$ python py.py nazwa_pliku --ilosc test
usage: py.py [-h] [-a ARGUMENT] [-i ILOSC] filename
py.py: error: argument -i/--ilosc: invalid int value: 'test'

```

Argumenty zaczynające się jednym myślnikiem są skróconymi wersjami, podwójnym - pełne. Nie musimy definiować obu, wystarczy jedno oznaczenie, ale standardem jest obsługa krótkiej i rozszerzonej wersji.

#### Jak sprawdzić dostępne argumenty?

To jedna z ważniejszych właściwości tego modułu, na starcie, bez dodatkowej pracy, sam przygotuje dla nas magiczny argument *-h*. Pozwala on na wyświetlenie automatycznie wygenerowanej listy argumentów, wraz z ich opisami (które podaliśmy w parametrze *description*), typami oraz domyślnymi wartościami (parametr *default*) - czyli tak zwany *help*. Zobaczmy co się stanie gdy go użyjemy.

```
$ python py.py -h
usage: py.py [-h] [-a ARGUMENT] [-i ILOSC] filename

Tutaj możemy podać zwięzły opis naszego skryptu

positional arguments:
  filename              Opis tego argumentu (krótkie wyjaśnienie co przyjmuje)
                        - tutaj uznajmy że nazwę pliku

optional arguments:
  -h, --help            show this help message and exit
  -a ARGUMENT, --argument ARGUMENT
                        Nasz opcjonalny argument
  -i ILOSC, --ilosc ILOSC
                        Argument o konkretnym typie i wartością domyślną

```

## Podsumowanie

Zachęcam do zapoznania się z dokumentacją modułu [argparse](https://docs.python.org/3.5/library/argparse.html), ponieważ przykład który pokazałem jest najprostszym z możliwych. Moduł ten oferuje o wiele więcej, a gwarantuję, że jak zaczniesz stosować argumenty linii komend w swoich skryptach, ich jakość i łatwość obsługi wzrośnie diametralnie.
