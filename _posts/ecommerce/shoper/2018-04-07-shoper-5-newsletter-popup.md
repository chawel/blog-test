---
layout: post
title: Shoper 5 - Newsletter PopUp
modified: Jak stworzyć Newsletter PopUp i Exit PopUp do Shoper z obsługą ciastek
categories: ecommerce/shoper
description:
tags: [shoper, popup, javascript]
image:
  feature:
  credit:
  creditlink:
comments:
share:
date: 2018-04-07T10:20:42+02:00
---
Jak stworzyć PopUp zawierający wszelkiego rodzaju informacje (kod rabatowy, kontakt, obrazek), jak i formularz zapisu do Newslettera. PopUp działa z Shoper 5 (na skórce RWD) i korzysta z jQuery.

<!-- more -->
## Od autora, czyli refleksja nad PopUp'em (można pominąć)
Z nieukrywaną niechęcią piszę ten poradnik. Jestem wielkim przeciwnikiem *"wyskakującego g..."* na stronach, może dlatego, że pamiętam jeszcze czasy, kiedy niczym króliki mnożyły się otwarte okna przeglądarki po wejściu na jedną stronę, a ich zamykanie było istną syzyfową pracą - bo przy wejściu na kolejną znowu otwierały się 3 nowe w tle. Szybko pojawiły się rozwiązania (oczywiście nie w IE) blokujące *"wyskakujące okienka"*, ale cóż z tego, jak obecnie mamy popupy zasłaniające całą treść strony (dzięki JavaScript...). Niestety, rynek weryfikuje - jest zapotrzebowanie na tego typu skrypty. Nadal w przekonaniu ludzi od marketingu jest to skuteczne narzędzie do nakłonienia klienta do dokonania zakupu czy podania adresu email (a zwłaszcza do zatrzymania go "w ostatniej chwili", czyli przy próbie opuszczenia strony). Czy słusznie? Są tacy, jak ja, przeciwnicy "z zasady", i nie ma opcji dla akceptacji tego typu manewrów (działają na mnie jak czerwona płachta na byka). Są też statystyki i badania, które potwierdzają ich skuteczność:
* [Are Email Subscription Pop-ups Worth The Risk?](https://unbounce.com/email-marketing/get-subscribers-from-pop-ups/)
* [In Defense Of The Email Popup](https://conversionxl.com/blog/popup-defense/)

Tak czy inaczej, decyzje oraz sposób przedstawienia PopUp'u pozostawiam Tobie czytelniku, miej jednak na uwadze ludzi pokolenia Internet Explorer'a i Windowsa 98, którzy do dziś niczym weterani wojny w Wietnamie noszą nieodwracalne rany na duszy, które przy każdym "wyskoczonym" popupie, otwierają się na nowo, a traumy przeżytych horrorów tamtych czasów - powracają. 

<figure class="center">
	<img src='{{ site.url }}/images/gif/vet_flashback.gif' alt="">
	<figcaption>Flashback.</figcaption>
</figure>


## Najprostszy PopUp Newsletter
Zaczniemy powoli, od podstaw, aby zrozumieć co robimy, a Skrypt będzie ewoluował przez cały wpis. Utworzymy *moduł*, dzięki czemu nie musimy martwić się o to, że po aktualizacji, zostanie usunięty czy zmodyfikowany.

W *Panelu Administracyjnym* przechodzimy do zakładki *Styl graficzny & Nawigacja*, następnie wchodzimy w *Aktywny styl graficzny* i ostatecznie trafiamy do zakładki *Moduły* - tam wybieramy opcję *dodaj moduł*

<figure class="center">
	<img src='{{ site.url }}/images/shoper/shoper_module.gif' alt="">
	<figcaption>Dodawanie modułu.</figcaption>
</figure>

1. Definiujemy tytuł naszego modułu (może być dowolny - taki aby odróżnić go od reszty modułów) - proponuję `PopUp Main`
2. Ustaw *Obramowanie* na **NIE**
3. Dalej jest *HTML ID* - tutaj już nie ma takiej dowolności, więc aby nie komplikować, wpisz tam: `popup-main`
4. *Tryb* ustawiamy na **HTML**
5. Na dole w edytorze kliknij przycisk **wyłącz edytor** i w polu wklej kod

### Skrypt JavaScript
Ten krótki skrypt JS odpowiada za wyświetlenie popup'u.

{% highlight javascript %}
<script type="text/javascript">
// Czekamy na załadowanie się strony
document.onload = popupShow();

// Funkcja wyświetlająca PopUp
function popupShow() {
    // Sprawdzamy czy popup się pojawił, jeżeli tak do dodajemy do niego przycisk "zamknij"
    if( $("#popup-main").length) {
        $("#popup-main").prepend('<a id="popup-exit"><span>zamknij x</span></a>');

        // Wyświetlamy popup
        $("#popup-main").css( 'display', 'block' );
    }

    // Obsługa przycisku "zamknij" - czyli wyłącz popup
    $("#popup-exit").click(function() {
        $("#popup-main").remove();
    });

    // Obsługa przycisku "Zapisz Się", wtedy też popup wyłączymy
    $("#popup-form").submit(function(e) {
        // Zauważ, że można zamknąć popup, po prostu ukrywając go przez styl CSS
        $("#popup-main").css( 'display', 'none' );
    });
};
</script>
{% endhighlight %}

### Formularz Newsletter HTML
Prosty formularz zapisu do newslettera napisany w HTML. Doklejmy do naszego modułu, na sam dół.

{% highlight html %}
<div class="popup-containter">
    <h2>Zapisz się do newslettera!</h2>
    <form id="popup-form" action="/pl/newsletter/sign" method="post">
        <fieldset>
            <label>Poniżej wpisz swój adres e-mail, aby uzyskać rabat</label>
            <input name="email" size="30" type="email" />
            <button class="btn btn-red" type="submit" value="Zapisz Się">Zapisz Się!</button>
        </fieldset>
    </form>
</div>
{% endhighlight %}

### Styl CSS
Potrzebujemy jeszcze jakiegoś CSS, aby zdefiniować jak ma wyglądać PopUp. Przechodzimy do zakładki *Własny styl CSS* i w nim wklejamy nasz bardzo uproszczony styl

<figure class="center">
	<img src='{{ site.url }}/images/shoper/shoper_css.png' alt="">
</figure>

{% highlight css %}
/*---- Pop-UP ----*/
#popup-main{
  display: none;

  width: 480px;
  min-height: 100px;

  z-index: 99999;
  position: fixed;
  top: 100px;
  left: 50%;
  margin-left: -240px;

  background-color: white;
  border: 2px solid red;
  box-shadow: 0 0 0 2000px rgba(0,0,0,0.5);
  padding: 10px;
}

#popup-main a#popup-exit{
  background-color: red;
  position: absolute;
  right: 0;
  top: 0;
  padding: 5px 10px;
  color: white;
  cursor: pointer;
}

#popup-main .popup-containter {
  text-align: center;
}
{% endhighlight %}

### Umieszczenie PopUp na stronie głównej

Aby wyświetlić stworzony moduł `PopUp Main`, musimy go gdzieś umieścić (możemy umieszczać moduły na konkretnych podstronach, np. tylko na stronie produktu albo tylko na stronie głównej). W naszym przypadku będzie wyświetlany od razu po wejściu na stronę główną (i tylko tam).

<figure class="center">
	<img src='{{ site.url }}/images/shoper/shoper_module_insert.gif' alt="">
	<figcaption>Wstawianie modułu.</figcaption>
</figure>

### Efekt końcowy

Jeżeli trzymaliście się instrukcji, po wejściu na stronę główną sklepu, powinien wyświetlić się taki PopUp.

<figure class="center">
	<img src='{{ site.url }}/images/shoper/popup_newsletter.png' alt="">
	<figcaption>Nasz PopUp na stronie głównej.</figcaption>
</figure>

## Pliki cookie w JS
Jak pewnie zauważyłeś, nasz *prostacki* PopUp wcale nie daje za wygraną i wyskakuje za każdym razem jak odświeżymy stronę (nawet jeżeli go poprzednio zamknęliśmy). Nie tylko zamknięcie go nie spowoduje jego permanentnego ukrycia, ale nawet jeżeli odwiedzający nasz sklep złamie się pod naporem takiej siły perswazji i zapisze się do newslettera - PopUp nadal będzie ~~napier....~~ przy każdym wejściu na stronę główną.

<figure class="center">
	<img src='{{ site.url }}/images/kapitan.jpg' alt="kapitan bomba">
	<figcaption>źródło: Git Produkcja.</figcaption>
</figure>

Musimy jakoś rozróżnić, czy odwiedzający widział już PopUp i ukryć go na pewien okres czasu (dzień, tydzień, miesiąc, rok), a nawet zupełnie go dezaktywować w przypadku wypełnienia formularza newslettera. 

Jak to zrobić? Wykorzystamy mechanizm ciasteczek, czyli tzw. *cookies*. Są to pliki (tekstowe), które zawierają w sobie informacje zapisywane przez przeglądarkę dla odwiedzanej witryny. Przy każdych odwiedzinach tej strony, nasza przeglądarka udostępnia zapisany przez nią plik cookie, z którego może odczytać pewne dane - to tak w skrócie.

Zaimplementujmy do naszego PopUp'a mechanizm zapisu i odczytu ciastek. Założenie są takie: 
* Pierwsza wizyta - wyświetl popup,
* Zamknięcie popupu - nie pokazuj przez 24 godziny, 
* Zapis do newslettera - nie pokazuj przez rok (365 dni)

Nie ukrywam swojej sympatii do Pythona i jego *ZEN* - wyznaję więc pogląd, że po co wymyślać na nowo koło, skoro są gotowe rozwiązania. Gotowe funkcje do zapisu i odczytu ciastek znalazłem na stronie: [JavaScript - Cookies](https://www.quirksmode.org/js/cookies.html)

{% highlight javascript %}
function setCookie(name,value,days) {
    var expires = "";
    if (days) {
        var date = new Date();
        date.setTime(date.getTime() + (days*24*60*60*1000));
        expires = "; expires=" + date.toUTCString();
    }
    document.cookie = name + "=" + (value || "")  + expires + "; path=/";
}

function getCookie(name) {
    var nameEQ = name + "=";
    var ca = document.cookie.split(';');
    for(var i=0;i < ca.length;i++) {
        var c = ca[i];
        while (c.charAt(0)==' ') c = c.substring(1,c.length);
        if (c.indexOf(nameEQ) == 0) return c.substring(nameEQ.length,c.length);
    }
    return null;
}
{% endhighlight %}

Nazwy funkcji wymowne, wiadomo co od czego. Zainteresowanych co konkretnie funkcje te robią, odsyłam do [strony źródłowej](https://www.quirksmode.org/js/cookies.html), gdzie wyczerpano temat (wytłumaczona każda linijka kodu). Ja skupie się na ich użyciu. 

Dodajmy więc funkcję, która sprawdzi czy istnieje ciastko, a jeżeli nie, to wyświetli popup.

{% highlight javascript %}
function checkCookie() {
	var popupCookie = getCookie("popup");
	if (popupCookie == null || popupCookie == "") {
	    popupShow();
	}
}
{% endhighlight %}

A teraz cały skrypt spełniający nasze założenia.
{% highlight javascript %}
<script type="text/javascript">
// Czekamy na załadowanie się strony
document.onload = checkCookie();

// Funkcja zapisująca cookie
function setCookie(name,value,days) {
    var expires = "";
    if (days) {
        var date = new Date();
        date.setTime(date.getTime() + (days*24*60*60*1000));
        expires = "; expires=" + date.toUTCString();
    }
    document.cookie = name + "=" + (value || "")  + expires + "; path=/";
}

// Funkcja odczytująca cookie
function getCookie(name) {
    var nameEQ = name + "=";
    var ca = document.cookie.split(';');
    for(var i=0;i < ca.length;i++) {
        var c = ca[i];
        while (c.charAt(0)==' ') c = c.substring(1,c.length);
        if (c.indexOf(nameEQ) == 0) return c.substring(nameEQ.length,c.length);
    }
    return null;
}

// Funkcja decydująca czy wyświetlić popup czy nie (na podstawie cookies)
function checkCookie() {
	var popupCookie = getCookie("popup");
	if (popupCookie == null || popupCookie == "") {
	    popupShow();
	}
}

// Funkcja wyświetlająca PopUp
function popupShow() {
    if( $("#popup-main").length) {
        $("#popup-main").prepend('<a id="popup-exit"><span>zamknij x</span></a>');
        $("#popup-main").css( 'display', 'block' );
    }

    $("#popup-exit").click(function() {
        $("#popup-main").remove();

        // Popup zamknięty przez kliknięcie przycisku "Zamknij"
        // Zapisujemy do ciastka, wartość 'hide' w kluczu 'popup' z terminem ważności 1,
        // aby nie wyświetlał się przez 1 dzień
        setCookie("popup", "hide", 1);
    });

    $("#popup-form").submit(function(e) {
        $("#popup-main").css( 'display', 'none' );

        // Popup zamknięty przez podanie adresu e-mail do newslettera
        // Zapisujemy do ciastka, aby nie wyświetlał się przez 365 dni
        setCookie("popup", "hide", 365);
    });
};
</script>
{% endhighlight %}


## Exit Newsletter PopUp (przy próbie opuszczenia strony)
Ok, podstawy już za nami, więc trochę podkręćmy tempo i poziom. Zrobimy manewr, który potrafi skutecznie (podobno) zatrzymać odwiedzających na naszej stronie, przy próbie jej opuszczenia. PopUp wyświetli się jedynie wtedy, gdy kursor figuranta odjedzie poza obszar strony (czyli gdzieś w okolice paska adresu przeglądarki). 

**Pamiętaj, że nie jest to poradnik SEO czy internetowego marketingu**, ale osobiście uważam, że to rozwiązanie może wzbudzić kontrowersje, zwłaszcza, jeżeli klient przeszedł cały proces zakupowy bez próby opuszczenia strony, a nagle po złożeniu zamówienia, chcąc zamknąć przeglądarkę, pojawi mu się popup z kodem rabatowym.

<figure class="center">
	<img src='{{ site.url }}/images/zonk.jpg' alt="">
	<figcaption>No i zonk. <a href="https://www.tuwroclaw.com/wiadomosci,wroclawski-slownik-slangu-litera-z,wia5-3266-3428.html">źródło</a></figcaption>
</figure>

Wracając do tematu. Potrzebujemy obsługi tego zdarzenia (próba opuszczenia strony) - do naszego kodu wyżej dodamy kolejne funkcje i zmodyfikujemy zdarzenie `document.onload`:

{% highlight javascript %}
// Funkcja pomocnicza dla obsługi zdarzeń
// Przyjmuje 3 argumenty
// obj - obiekt którego zdarzenia będziemy wyłapywać
// evt - skrót od event, czyli na jakie konkretnie zdarzenie czekamy
// fn - funkcja którą uruchomimy, przy wykryciu zdarzenia
function addEvent(obj, evt, fn) {
    if (obj.addEventListener) {
        obj.addEventListener(evt, fn, false);
    }
    else if (obj.attachEvent) {
        obj.attachEvent("on" + evt, fn);
    }
}


// Czekamy na załadowanie się strony
// Dodajemy obsługę zdarzenia
// * document - to interfejs reprezentujący całą naszą stronę
//              załadowaną przez przeglądarkę (zawiera tzw. drzewo DOM)
// * mouseout - zdarzenie wyjechania poza obszar obiektu, w tym przypadku
//              obszar to document - czyli cała strona
//
document.onload = addEvent(document, "mouseout", function(e) {
    // Ta linijka służy kompatybilności z IE
    // 'e' oznacza wywołane zdarzenie, normalnie powinno zostać przekazane
    // ale Microsoft robi to inaczej i musimy pobrać je z 'window.event'
    // konkretniejsze wyjaśnienie: https://www.quirksmode.org/js/events_access.html
    e = e ? e : window.event;
    var from = e.relatedTarget || e.toElement;
    if (!from || from.nodeName == "HTML") {
        // Jeżeli kursor poza obszarem okna strony 
        // - sprawdź ciastka i zdecyduj czy wyświetlić popup
        checkCookie();
    }
});
{% endhighlight %}

## Podsumowanie
Przedstawione rozwiązanie jest dość prostackie, należałoby pewnie:
* Rozbudować styl CSS (dostosować do mobilnych urządzeń)
* Dodać funkcję do walidacji wpisanego adresu e-mail (aby np. tylko poprawny adres był brany pod uwagę)
* Może użyć efektów (animacji) oraz opóźnienia pojawiania się
* Zastosować dodatkowe założenia:  
  - jeżeli klient próbuje "porzucić koszyk" - wyświetl te produkty w PopUp i nadaj dodatkowy rabat  
  - jeżeli jest to klient który dokonał już zakupu - nie wyświetlaj PopUp'a z rabatem a tylko z zapisem do newslettera  
  - umieszczenie PopUp tylko w konkretnej sekcji sklepu - np. tylko na liście produktów)

Ktoś zapyta, to dlaczego nie opisałem jak takie założenia zaimplementować? Proste - ten artykuł ma mieć charakter dydaktyczny, skupiłem się więc na podstawach i najprostszych rozwiązaniach, bez wodotrysków, aby nie komplikować i pozwolić Wam zrozumieć co się dzieje. 

Jeżeli ktoś szukał gotowego PopUp i trzymał już palce na `Ctrl` + `c` to wiem, że taką osobę zawiodłem. Jednak szukających pomocy w zrozumieniu podstaw JS i mechanizmu modalnego PopUp, mam nadzieję że zaspokoiłem - tych zapraszam do komentowania i zadawania pytań, na które z chęcią odpowiem.

