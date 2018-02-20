---
layout: post
title: PyCharm - uruchamianie skryptów z parametrami (argumentami)
modified:
categories: python/tips
description: 
tags: [pycharm, tips, python]
image:
  feature: pycharm_logo.jpg
  credit: JetBrains
  creditlink: https://jetbrains.com/pycharm/
comments:
share:
date: 2018-02-20T19:34:25+01:00
---

## Szybki Tip

Niedawno omawiałem w [tym wpisie]({{ site.baseurl }}{% post_url /python/tutorial/2018-02-19-python-argparse-przekazywanie-parametrow-argumentow-uruchomieniowych-do-skryptu %}), jak dodać obsługę argumentów linii komend naszych skryptów w Python. Na szybko postaram się pokazać w jaki sposób możemy odpalać skrypty, przekazując argumenty (parametry) wywołania w [PyCharm](https://jetbrains.com/pycharm/). Sprawa jest banalnie prosta.

<!-- more -->

## Profile uruchamiania skrytpu

W prawym górnym rogu głównego okna PyCharm'a, odnajdziemy znajomo wyglądające kontrolki (Run, Debug, Stop). Po ich lewej mamy rozwijaną listę konfiguracji.

<figure class="center">
	<img src='{{ site.url }}/images/pycharm/pycharm_toolbar.gif' alt="">
	<figcaption>Kontrolki na toolbarze</figcaption>
</figure>

Na samej górze tej mini-listy, jest opcja *Edit Configuration*. Otwiera ona okno, w którym widzimy możliwość podania *Parameters*, to tutaj deklarujemy nasze argumenty z którymi wywoła się nasz skrypt.

<figure class="center">
	<img src='{{ site.url }}/images/pycharm/pycham_edit_config.gif' alt="">
	<figcaption>Karta konfiguracyjna</figcaption>
</figure>

Gdyby zdarzyło się, że na liście konfiguracji nie ma nazwy naszego skryptu - wystarczy go przynajmniej raz odpalić. Automatycznie powinna wygenerować się standardowa karta konfiguracyjna dla tego skryptu, dzięki czemu nie będziemy musieli wypełniać jej sami.

<figure class="center">
	<img src='{{ site.url }}/images/pycharm/pycham_no_project.gif' alt="">
	<figcaption>I już jest</figcaption>
</figure>

Na zakończenie, polecam zapoznać się z innymi opcjami w profilu uruchamiania, niektóre jeszcze opiszę w następnych wpisach, a te bardziej oczywiste zachęcam przetestować samemu.
