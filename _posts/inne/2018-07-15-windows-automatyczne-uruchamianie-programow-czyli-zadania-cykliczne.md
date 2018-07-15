---
layout: post
title: Windows automatyczne uruchamianie programów, czyli harmonogram zadań cyklicznych
modified:
categories: inne
description: Jak ustalić automatyczne uruchamianie programów lub innych zadań w systemie Windows - Task scheduler
tags: [windows, tips]
image:
  feature: task_scheduler.jpg
  credit:
  creditlink:
comments:
share:
date: 2018-07-15T19:51:06+02:00
---
Nie chodzi o autostart, który uruchamia programy wraz ze startem systemu. Zaprezentuje narzędzie Task scheduler (Harmonogram zadań), które jest odpowiednikiem Linux'owego CRON na systemach Windows.

<!-- more -->

# Wstęp
Systemy z rodziny Unix oferują usługę CRON, która służy do harmonogramowania zadań o określonych porach. Wspominam o nim, ponieważ tak jak wiele osób wie o istnieniu CRON, tak niestety często spotykam się z pytaniem o jego odpowiednik w systemach Windows. Przedstawię narzędzie **Task scheduler** - które niestety jest dość mało wyeksponowane w systemie firmy Microsoft, a szkoda, bo jest naprawdę użyteczne oraz oferuje wiele przydatnych funkcji,

# Zadania cykliczne
Na przykładzie prostego scenariusza (codzienne uruchamianie programu o danej godzinie) pokażę, jak używać narzędzia **Task scheduler** na systemie **Windows 10**.

## Uruchomienie
Zacznijmy od znalezienia narzędzia, co nie jest tak oczywiste jak powinno być. Jest kilka sposobów, jeden to wpisanie w "szukajce" menu START frazy "Task scheduler" (pol. **"Harmonogram zadań"**).

Jednak łatwiej jest (bo bez względu na język systemu) zrobić to w ten sposób:

- Otwieramy okno "uruchom" (skrót klawiszowy to: <kbd>Win</kbd> + <kbd>R</kbd>, gdzie klawisz <kbd>Win</kbd> to ten na klawiaturze z logiem Windows)
<figure class="center">
	<img src='{{ site.url }}/images/scheduler/run.png' alt="">
	<figcaption>Run...</figcaption>
</figure>
- Wpisujemy w nim: `taskschd.msc` i klikamy **OK**
- Powinno pojawić się okno usługi Task Scheduler
<figure class="center">
	<img src='{{ site.url }}/images/scheduler/scheduler.png' alt="">
	<figcaption>Harmonogram zadań</figcaption>
</figure>

## Ustawienie cyklicznego zadania
Dobrym rozwiązaniem tego narzędzia jest kreator zadań, który przeprowadzi nas krok po kroku.

- Po prawej stronie wybieramy *Create Basic Task...* i pojawi się okno kreatora zadań
- W polu *Name* wpisujemy nazwę naszego zadania, a w polu *Description* jego opis (opcjonalny) i przechodzimy do następnego kroku klikając **Next**
<figure class="center">
	<img src='{{ site.url }}/images/scheduler/basic1.png' alt="">
	<figcaption>Create Basic Task...</figcaption>
</figure>
- W kolejnym kroku wybieramy kiedy ma wywołać się dane zadanie, skoro ma wywoływać się codziennie wybieramy opcję *Daily* (czyli każdego dnia) i przechodzimy dalej
<figure class="center">
	<img src='{{ site.url }}/images/scheduler/basic2.png' alt="">
	<figcaption>Cykliczność uruchamiania</figcaption>
</figure>
- Następnie wybieramy datę od kiedy ma zacząć się cykl wywoływania tego zadania - pole *Start*, w naszym przypadku będzie to od dnia dzisiejszego, oraz godzinę w której dane zadanie zostanie uruchomione, można również ustawić aby wykonywało się co określoną liczbę dni (np. co drugi dzień, co trzeci itd.), ale w naszym przypadku zostawiamy w polu *Recur every* wartość 1 - co oznacza powtarzanie się zadania codziennie.
<figure class="center">
	<img src='{{ site.url }}/images/scheduler/basic3.png' alt="">
	<figcaption>Harmonogram zadania</figcaption>
</figure>
- Dalej mamy wybór co dokładnie ma się wydarzyć, wybieramy *Start a program* (będzie uruchamiał się wskazany program)
<figure class="center">
	<img src='{{ site.url }}/images/scheduler/basic4.png' alt="">
	<figcaption>Zadanie do wykonania</figcaption>
</figure>
- W kolejnym etapie wybieramy program który ma się uruchomić, klikamy przycisk *Browse...* i wskazujemy plik .exe programu który ma zostać odpalony
<figure class="center">
	<img src='{{ site.url }}/images/scheduler/basic5.png' alt="">
	<figcaption>Wybór programu do uruchomienia</figcaption>
</figure>
- Ostatnim etapem jest potwierdzenie i sprawdzenie czy to co *"przeklikaliśmy"* zgadza się z tym co jest w podsumowaniu, jeżeli tak, to klikamy *Finish* i w tym momencie zostanie dodany nowe zadanie cykliczne
<figure class="center">
	<img src='{{ site.url }}/images/scheduler/basic6.png' alt="">
	<figcaption>Podsumowanie kreatora</figcaption>
</figure>
- Nasz *"task"* możemy sprawdzić w zakładce **Task Scheduler Library**
<figure class="center">
	<img src='{{ site.url }}/images/scheduler/task.png' alt="">
	<figcaption>Biblioteka zadań</figcaption>
</figure>

# Podsumowanie
Jak widać nie jest to nic trudnego, a z pewnością ułatwi to życie ludziom leniwym lub zapominalskim. Jak zwykle, zachęcam do samodzielnego odkrywania pozostałych możliwości tego narzędzia.
