---
layout: post
title: Google SpreadSheets i Power Tools - Jak wydzielić produkty główne z wariantów?
modified:
categories: inne
description: Na przykładzie pliku z hurtowni dywanów, pokażę proces wydzielenia produktów głównych z wariantów.
tags: [excel, spreadsheets, powertools, tips]
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2018-05-13T19:12:10+02:00
---

Tym wpisem chcę zacząć serię mini-poradników w których będę opisywał swoje doświadczenia i patenty związane z pracą z plikami zawierającymi dane. W tym poście opiszę dość częsty problem, kiedy dostajemy od dostawcy plik w pojedynczymi produktami, które tak na prawdę są wariantami jednego produktu.

<!-- more -->

# Wstęp

Obecnie standardem jest, że platformy sprzedażowe oferują obsługę wariantów produktów. Jest to bardzo wygodne rozwiązanie dla kupujących, bo na karcie produktu mogą zobaczyć dostępne warianty oraz wybrać który chcą zamówić, zamiast przeszukiwać asortyment by odnaleźć np. wariant tej bluzki w kolorze białym. Wystawiając wszystkie kolory jednej bluzki jako osobne produkty, bardzo szybko skończyły by się nam zasoby serwera (miejsce i pojemność bazy danych). Dlatego, w takim przypadku staramy się zwykle dodawać 1 produkt główny (np. bluzka) i do niego dodawać warianty (np. kolor biały, kolor czarny), nie powielając przy tym tego samego opisu i parametrów.

Jeżeli *wciągamy* asortyment z zewnętrznej hurtowni, otrzymujemy od nich plik z danymi (dla potrzeby artykułu uznajmy, że w formacie CSV). Większość hurtowni trzyma każdy wariant jako osobny produkt, więc zwykle otrzymujemy plik o podobnej strukturze do tej:
```
kod produktu - bluzka biała - opis - cena
kod produktu - bluzka czarna - opis - cena
kod produktu - bluzka niebieska - opis - cena
...
```

W takiej sytuacji najczęściej kopiujemy pierwsze wystąpienie danego produktu i na jego podstawie wprowadzamy produkt główny, a następnie do niego (produktu głównego) dodajemy jeszcze warianty.

# Google SpreadSheets
Znacie zapewne **Excel** z pakietu *MS Office*, to program do pracy z danymi w formie tabel. *Google* wprowadziło swój odpowiednik *pakietu Office*, a **[SpreadSheets](https://docs.google.com/spreadsheets/u/0/)** jest właśnie jak Excel. **Całość jest darmowa** i działa w tzw. chmurze, bez instalacji, a pliki nad którymi pracujemy zapisywane są na naszym Dysku Google.

## Wtyczki i dodatki - Power Tools
*Googlowy Office* oferuje również szeroką gamę dodatków i wtyczek. Jednym z nich jest *kombajn* o nazwie **Power Tools** - z nim otwierają się zupełnie nowe możliwości pracy z tabelami.

## Gdzie haczyk?
Niestety, muszę uprzedzić, są minusy:
- Power Tools oferuje **30 dniową wersję próbną**, później albo kupujemy roczną subskrypcję *(~40-30 $)*
- Korzystanie z Power Tools wiąże się z udzieleniem mu niezbędnych uprawnień i wyrażeniem zgody na pewne warunki. Power Tools prosi o umożliwienie łączenia się z zewnętrznym serwisem - czyli jest ryzyko, że nasze dane mogą być przetwarzane na innym serwerze, a to oznacza, że **NIE POLECAM używania tej wtyczki do wrażliwych danych (np. dane osobowe)**.
- Google SpreadSheets to usługa w chmurze i jeżeli również ze strony Googla obawiamy się *kompromitacji* danych, to należy się zastanowić na używaniem ich *pakietu biurowego*

## Warto?
**Osobiście uważam, że warto**, zwłaszcza jeżeli obrabiamy dane które i tak upubliczniamy (czyli asortyment sklepu). Dodatkowo zyskujemy możliwość pracy z naszymi dokumentami na każdym urządzeniu z przeglądarką internetową (np. komórka, tablet).

*Google Sheets* pozwala pracować na jednym dokumencie z innymi osobami w tym samym momencie oraz udostępniać je w prosty sposób. W kwestiach funkcjonalności - do podstawowych operacji (czyli 90% przypadków użycia) jest wystarczający. Jeżeli czegoś nam brakuje, to przy użyciu jego prostego systemu dodawania i zarządzania dodatkami, łatwo dobierzemy zestaw dodatkowych narzędzi.

## Jak zacząć pracę z Google SpreadSheets
Jeżeli nie posiadamy konta *Google*, to należy je założyć, a jak mamy - to się logujemy i przechodzimy do [Google SpreadSheets](https://docs.google.com/spreadsheets/u/0/).

Aby otworzyć lokalny plik, klikamy na ikonkę folderu
<figure class="center">
	<img src='{{ site.url }}/images/googlespreadsheets/ss1.jpg' alt="">
	<figcaption>Panel Google SpreadSheets</figcaption>
</figure>

Następnie w wyświetlonym oknie wybieramy zakładkę *Upload* i możemy przeciągnąć i upuścić plik albo wybrać go z dysku po kliknięciu przycisku *Select a file from your computer*
<figure class="center">
	<img src='{{ site.url }}/images/googlespreadsheets/ss2.jpg' alt="">
	<figcaption>Otwieranie pliku z komputera</figcaption>
</figure>


# Instalacja wtyczki Power Tools
Kiedy już otworzymy nasz plik, zobaczymy znajomy interfejs. Duża tabela i menu kontekstowe na górze z narzędziami. W menu na górze wybieramy *Add-ons* > *Get add-ons*, w otwartym oknie w wyszukiwarce wpisujemy *power tools* i potwierdzamy (Enter). Na liście znajdujemy **Power Tools** i klikamy przycisk **+FREE**. Teraz, zostaniemy przedstawieni na jakich zasadach i uprawnieniach działa wtyczka, aby jej używać musimy się zgodzić.

<figure class="center">
	<img src='{{ site.url }}/images/googlespreadsheets/addon2.gif' alt="">
	<figcaption>Proces instalacji dodatku Power Tools</figcaption>
</figure>

# Obróbka danych z hurtowni
Zaczynamy otwierając plik od hurtowni (w tym przypadku CSV) w *GSSheets*. Moje przykładowe dane wyglądają tak:
<figure class="center">
	<img src='{{ site.url }}/images/googlespreadsheets/example_data.jpg' alt="">
	<figcaption>Przykładowy plik CSV z hurtowni</figcaption>
</figure>

## Przykład
Na przykładzie pliku z hurtowni dywanów, pokażę proces wydzielenia produktów głównych. 

Musimy zdecydować jaką strategię obierzemy. W moim przypadku sprawa wydaję się prosta, kolumna SKU zawiera kody produktów. Zauważ, że kod produktu składa się z 3 części oddzielonych myślnikami. Co więcej, widzimy że pewne części tego kodu są wspólne dla wariantów tego samego produktu, zacznę więc od wydzielenia ich.

<figure class="center">
	<img src='{{ site.url }}/images/googlespreadsheets/tools.jpg' alt="">
	<figcaption>Lista produktów głównych</figcaption>
</figure>

### Wydzielanie części kodu

1. Odpalamy dodatek *Power Tools* ( *Add-ons > Power Tools > Start* )
2. Zaznaczamy naszą kolumnę **SKU**
3. W menu klikamy na *Split* > wybieramy opcję *Split values by characters* > w niej oznaczamy jedynie *Custom* > w polu tekstowym obok podajemy znak "-" *(myślnik)*
4. Zatwierdzamy przyciskiem **Split**

{% capture images %}
	/images/googlespreadsheets/split.jpg
	/images/googlespreadsheets/split3.jpg
{% endcapture %}
{% include gallery images=images caption="" cols=2 %}

Stworzone zostały 3 nowe kolumny z wartościami, tę z cząstką kodu który się powtarza nazwijmy *SKU-main*.

### Znajdowanie duplikatów

Wiemy już, że wszystkie rozmiary danego produktu łączy ten sam kod (który wydzieliliśmy), a my chcemy uzyskać listę ich produktów głównych. Wydzielmy więc unikalne wystąpienia tego kodu.

1. Odpalamy dodatek *Power Tools* ( *Add-ons > Power Tools > Start* )
2. W menu klikamy na *Data* > następnie *Remove duplicates* > pojawi się okno, które krok po kroku pomoże nam zdefiniować jak chcemy żeby ten proces przebiegł
3. Pierwszy krok to wskazanie obszaru działania, klikamy na *Auto select* (zaznaczy wtedy cały arkusz) > klikamy *Next*
4. W drugim kroku mamy do wyboru 4 opcje, my wybieramy **Uniques + 1st occurences**:
- **Duplicates** - wyszukuje jedynie duplikaty, bez pierwszego wystąpienia
- **Duplicates + 1st occurences** - wyszukuje duplikaty, wraz z ich pierwszym wystąpienia
- **Uniques** - wyszukuje jedynie wartości które się nie powtarzają
- **Uniques + 1st occurences** - wyszukuje wartości unikalne, wraz z ich pierwszymi wystąpieniami wartości zduplikowanych
5. Krok trzeci to wybór kolumny którą będziemy przeszukiwać w poszukiwaniu duplikatów, zaznaczamy kolumnę **SKU-main**
6. Ostatni krok pozwala zdefiniować co z tymi danymi zrobimy, nas interesuje opcja *Copy to another location > New Sheet* (skopiuje dane do nowego arkusza)
7. Zatwierdzamy przyciskiem **Finish**

{% capture images %}
	/images/googlespreadsheets/dup1.jpg
	/images/googlespreadsheets/dup2.jpg
	/images/googlespreadsheets/dup3.jpg
	/images/googlespreadsheets/dup4.jpg
{% endcapture %}
{% include gallery images=images caption="Funkcja Find Duplicates" cols=2 %}

Do naszego pliku został dodany nowy arkusz, a w nim znajdują się pojedyncze wystąpienia produktów, czyli lista produktów głównych.
<figure class="center">
	<img src='{{ site.url }}/images/googlespreadsheets/dup_final.jpg' alt="">
	<figcaption>Lista produktów głównych</figcaption>
</figure>

# Podsumowanie
Mam nadzieję, że ten krótki wpis pomoże komuś w ogarnięciu plików z danymi. Oczywiście jest wiele innych możliwości i przypadków w których pomocne jest narzędzie Power Tools. Jeżeli masz nietypowy problem z danymi tabelarycznymi, opisz go w komentarzu, postaram się pomóc.



