---
layout: post
title: Skrypt do łączenia wielu zdjęć w jedno (Galeria Grid)
modified:
categories: python/scripts
description: Skrypt łączy zdjęcia z podanego katalogu w jeden obraz (idealne rozwiązanie dla miniatur aukcji wielowariantowych)
tags: [script, python, galeria, miniatura]
image:
  feature: imgmerge.jpg
  credit:
  creditlink:
comments:
share:
date: 2018-02-11T15:22:29+01:00
---

Pierwszy wpis z serii gotowy skrypt do automatyzacji konkretnej czynności. Ten, pomaga generować "miniatury", łącząc obrazy wielu przedmiotów na jednym "płótnie" (tak będę nazywał obszar obrazu naszej galerii). Pomocne dla osób, które chcą umieścić na aukcji, karcie produktu, zdjęcie poglądowe wszystkich wariantów (np. wielu kolorów, wzorów).
<!-- more -->

## Plan działania

Skrypty są idealne do realizowania określonych zadań, a więc musimy określić co konkretnie potrzebujemy zautomatyzować.

### Zadania:
1. Zebranie informacji o wszystkich obrazkach (ilość i wymiary)
2. Ustalenie optymalnych wymiarów miniatury na płótnie, uwzględniając
   * Ilość wszystkich obrazków
   * Ilość obrazków w jednej linii
   * Szerokość płótna
3. Modyfikacja obrazka i wklejenie go na płótno
   * Dla każdego obrazka
4. Zapis wyniku do pliku (w tym przypadku PNG)

## Skrypt

### Ważne! Każdy skrypt piszę w najnowszej wersji Python (3.5 i wyżej).

Potrzebować będziemy następujące biblioteki:

{% highlight python %}
import os
from PIL import Image
from math import ceil
{% endhighlight %}

### Ad. 1
Funkcja zbiera informacje o plikach w podanym folderze. Buduje "słownik" (dict) który użyjemy aby zachować informacje o ilości plików oraz ich wymiarach.

{% highlight python %}
def get_images_info(path):
    images = {}
    for root, dirs, files in os.walk(path):
        for name in files:
            with Image.open(os.path.join(root, name)) as temp_img:
                images[name] = temp_img.size

    return images
{% endhighlight %}

Przykład co zwraca funkcja:
{% highlight python %}
{'koszulka-czarna.jpg': (600, 600), 'main.png': (760, 376), 'koszulka-meska-oliwkowa.jpg': (308, 308), 'koszulka-oliwkowa.jpg': (308, 308), 'koszulka-oliwkowa (copy).jpg': (308, 308), 'koszulka-czarna (copy).jpg': (600, 600), 'koszulka-meska-oliwkowa (copy).jpg': (308, 308)}
{% endhighlight %}

### Ad. 2
Korzystając z informacji z punktu 1, możemy wyliczyć średnie (optymalne) wymiary dla obszaru jednego obrazka na płótnie. Najlepiej kiedy obrazki nie odbiegają znacząco wymiarami od siebie (przykład kłopotliwego zestawu: 64x64, 1200x3000, 800x12000).

{% highlight python %}
def get_standard_size(size_list):
    count = len(size_list)
    standard_x = 0
    standard_y = 0

    for size in size_list:
        standard_x += size[0]
        standard_y += size[1]
    else:
        standard_x = standard_x / count
        standard_y = standard_y / count

    return standard_x, standard_y
{% endhighlight %}

Pozostałe potrzebne dane wczytamy "po drodze"

### Ad. 3
Czas na główną funkcję, która zrobi robotę.

Funkcja pomocnicza, do wczytywania obrazków:

{% highlight python %}
def open_image(file_path):
    with Image.open(file_path) as current_image:
        temp_img = current_image.copy()

    return temp_img
{% endhighlight %}

Danie główne:

{% highlight python %}
def merge_images(path, canvas_width=760, in_row=5, output_file="main.png"):
    img_dict = get_images_info(path)

    if not img_dict:
        raise BaseException("Couldn't gather information about images!")

    img_count = len(img_dict.keys())

    standard_size = get_standard_size(img_dict.values())

    space_between = 2

    thumb_offset_y = 0
    thumb_offset_x = 0

    img_index = 0
    temp_img_index = 0

    if img_count < in_row:
        thumb_x = int(canvas_width / img_count - space_between)
        width_percent = (float(thumb_x) / float(standard_size[0]))
        thumb_y = int((float(standard_size[1]) * float(width_percent)))
        canvas_height = thumb_y + space_between
    else:
        thumb_x = int(canvas_width / in_row - space_between)
        width_percent = (float(thumb_x) / float(standard_size[0]))
        thumb_y = int((float(standard_size[1]) * float(width_percent)))
        canvas_height = int(ceil(img_count / in_row) * thumb_y)

    main_img = Image.new('RGB', (canvas_width, canvas_height), (255, 255, 255, 255))

    for img_filename in img_dict.keys():
        original_img = open_image(os.path.join(path, img_filename))

        thumb_img = original_img.resize((thumb_x, thumb_y), Image.ANTIALIAS)

        if img_index >= in_row:
            thumb_offset_y += thumb_y + space_between
            thumb_offset_x = 0
            temp_img_index += img_index
            images_left = img_count - temp_img_index

            if (img_count - temp_img_index) < in_row:
                thumb_offset_x = ((canvas_width - (2*space_between)) - (images_left * thumb_x)) / 2

            img_index = 0

        offset = (int(thumb_offset_x), int(thumb_offset_y))
        main_img.paste(thumb_img, offset)

        img_index += 1
        thumb_offset_x += thumb_x + space_between

    main_img.save(os.path.join(path, output_file), "PNG")
    main_img.close()
{% endhighlight %}

Dobra, time out, spróbuję trochę wyjaśnić co się dzieje na górze.

{% highlight python %}
# dla ułatwienia, tylko podanie ścieżki jest wymagane, reszta argumentów jest opcjonalna
# i posiada przypisane wartości domyślne
def merge_images(path, canvas_width=760, in_row=5, output_file="main.png"):
    # używamy naszej funkcji aby zebrac informację o wszystkich obrazkach w podanej ścieżce
    img_dict = get_images_info(path)

    # małe zabezpieczenie na wypadek gdyby nie udało się wczytać informacji o plikach 
    # (np brak plików w podanej ścieżce)
    if not img_dict:
        raise BaseException("Couldn't gather information about images!")

    # ilość obrazków
    img_count = len(img_dict.keys())

    # nasza funkcja przyjmuje listę, więc korzystamy z values()
    # które zwraca nam wartości danego słownika w formie listy
    # czyli wszystkie wartości rozmiarów [(x, y), ..., (x, y)]
    standard_size = get_standard_size(img_dict.values())

    # zmienna która określa odstęp między powierzchniami dla każdego obrazka na płótnie
    space_between = 2

    # wstępna deklaracja innych zmiennych
    thumb_offset_y = 0
    thumb_offset_x = 0

    img_index = 0
    temp_img_index = 0
{% endhighlight %}

W tym miejscu sprawdzamy czy ilość obrazków jest większa niż maksymalna ilość dla jednego rzędu. Jeżeli nie, nie uwzględnimy np. 5 miejsc w rzędzie, ale tyle ile realnie mamy obrazków.

{% highlight python %}
    if img_count < in_row:
        thumb_x = int(canvas_width / img_count - space_between)
        width_percent = (float(thumb_x) / float(standard_size[0]))
        thumb_y = int((float(standard_size[1]) * float(width_percent)))
        canvas_height = thumb_y + space_between
    else:
        thumb_x = int(canvas_width / in_row - space_between)
        width_percent = (float(thumb_x) / float(standard_size[0]))
        thumb_y = int((float(standard_size[1]) * float(width_percent)))
        canvas_height = int(ceil(img_count / in_row) * thumb_y)
{% endhighlight %}

Wyposażeni w niezbędne dane, tworzymy obszar galerii, czyli nasze przysłowiowe płótno.

{% highlight python %}
    main_img = Image.new('RGB', (canvas_width, canvas_height), (255, 255, 255, 255))
{% endhighlight %}

I konkretna robota, czyli otwieramy kolejne obrazki, zmniejszamy do wielkości obliczonej wcześniej, jeżeli jest to zdjęcie które nie zmieści się już w obecnym rzędzie przenosimy je na następny (pod spodem), wstawiając przy tym pomiędzy każdy obszar wolne miejsce, które zadeklarowaliśmy w zmiennej *space_between*.

{% highlight python %}
    for img_filename in img_dict.keys():
        original_img = open_image(os.path.join(path, img_filename))

        thumb_img = original_img.resize((thumb_x, thumb_y), Image.ANTIALIAS)

        if img_index >= in_row:
            thumb_offset_y += thumb_y + space_between
            thumb_offset_x = 0
            temp_img_index += img_index
            images_left = img_count - temp_img_index

            if (img_count - temp_img_index) < in_row:
                thumb_offset_x = ((canvas_width - (2*space_between)) - (images_left * thumb_x)) / 2

            img_index = 0

        offset = (int(thumb_offset_x), int(thumb_offset_y))
        main_img.paste(thumb_img, offset)

        img_index += 1
        thumb_offset_x += thumb_x + space_between
{% endhighlight %}

### Ad. 4

Nie zapominamy o zapisie i zamknięciu pliku

{% highlight python %}
    main_img.save(os.path.join(path, output_file), "PNG")
    main_img.close()
{% endhighlight %}

### To by było na tyle, cały skrypt z dodatną obsługą parametrów (co by odpalać z konsoli) dostępna na moim GitHub.

## Link do skrytpu:
[GitHub](https://github.com/chawel/imgmerge)


