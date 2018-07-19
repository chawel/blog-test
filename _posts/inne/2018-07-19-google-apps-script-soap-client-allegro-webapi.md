---
layout: post
title: Google Apps Script - SOAP Client - Allegro WebAPI
modified:
categories: inne
description: Zaawansowane użycie Google Apps Script, czyli klient dla Web Service Allegro.
tags: [soap, google apps script, allegro, web services]
image:
  feature: 
  credit: 
  creditlink:
comments:
share:
date: 2018-07-19T17:19:54+02:00
---

Przykład jak można komunikować się z Web Service (Allegro WebAPI) przez SOAP, a to wszystko w kilkuset linijkach kodu w Google Apps Script.

<!-- more -->

# Wstęp
Post obejmuje jedynie kwestie Google Apps Script, dodatkowo będziemy działać *"na żywca"* (bez biblioteki, klienta czy innego narzędzia które nam pomoże w komunikacji), więc potrzebna jest wiedza jak działa protokół SOAP.

Upewnij się, że przeczytałeś mój poprzedni post wprowadzający w temat obsługi WebAPI Allegro: [Podstawy obsługi Web API Allegro.pl - Web Services i moduł suds-jurko w Pythonie]({{ site.baseurl }}{% post_url /ecommerce/allegro/2018-03-09-podstawy-obslugi-web-api-allegro-pl-web-services-i-modul-suds-jurko-w-pythonie %}), jest tam przykład napisany w Pythonie, ale głównie chodzi o to abyś wiedział jak wygenerować klucz WebApi, jak przebiega proces uwierzytelniania oraz miał obycie z podstawami obsługi protokołu SOAP. 
Musisz również wiedzieć jak tworzyć skrypty w GAS - opisywałem podstawy w tym poście: [Google Apps Script - Wprowadzenie]({{ site.baseurl }}{% post_url /inne/2018-05-31-google-apps-script-wprowadzenie %})

Zakładam, że wiesz czym są API, Web service, protokół HTTP, Request, Response, XML oraz znasz podstawy JavaScript i wiesz czym jest Google Apps Script. To podstawy bez których niestety nie będziesz w stanie zrozumieć koncepcji zaprezentowanej w tym poście - a tłumaczenie ich mogłoby skończyć się nowym tomem książki o SOAP i Usługach sieciowych, zamiast zwięzłego postu o Google Apps Script.

**Jeżeli znasz już te zagadnienia, to zapraszam do lektury.**

# Google Apps Script
Zaczynamy od stworzenia nowego skryptu (poprzednio tworzyliśmy skrypt w obrębie arkusza kalkulacyjnego Google SpreadSheets, teraz skorzystamy w możliwości tworzenia samodzielnych skryptów - jednak nic nie stoi na przeszkodzie, aby sprzężyć skrypt z arkuszem). Przechodzimy do naszego panelu Google Apps Script (https://script.google.com/) i wybieramy w lewym górym rogu **+ Nowy Skrypt**.

<figure class="center">
	<img src='{{ site.url }}/images/gas/new_script.png' alt="">
	<figcaption>Nowy skrypt</figcaption>
</figure>

Teraz znajdujemy się w edytorze skryptów. To tutaj będziemy pisać naszą integrację z Allegro.

<figure class="center">
	<img src='{{ site.url }}/images/gas/script_editor.png' alt="">
	<figcaption>Edytor skryptu</figcaption>
</figure>

## XmlService
GAS oferuje szereg gotowych serwisów, które posiadają pewne funkcje. Do pracy z formatem XML wykorzystamy [XmlService](https://developers.google.com/apps-script/reference/xml-service/).
Posłuży nam on do stworzenia odpowiedniego zapytania, czyli dokumentu SOAP. 
Jak wspomniałem wcześniej, taki request zdefiniowany jest określoną strukturą, która musi zawierać obowiązkowe dla niej znaczniki. 

Zacznijmy więc od stworzenia *"pustego"* dokumentu SOAP - potraktuj to jak przygotowanie pustej koperty na wiadomość.

```
var soapIn = XmlService.parse('<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns1="https://webapi.allegro.pl/service.php"></SOAP-ENV:Envelope>');
```

Jak widzisz, nasza *"koperta"* definiuje jakieś `xmlns`, są to definicje przestrzeni nazw (ang. namespace, skrót: ns) - jest to trochę rozbudowany temat, więc na obecną chwilę zaufaj mi, że tak musi być. 
Skoro jest koperta to i musimy napisać wiadomość, którą chcemy wysłać. 
W tym celu stworzymy *"ciało"* naszego *request'a*, czyli element `Body`, będzie to taki list który włożymy do koperty.

```
// Pobieramy root node naszej koperty
var soapEnv = soapIn.getRootElement();
// Pobieramy jej namespace oznaczony jako 'SOAP-ENV'
var soapNS = soapEnv.getNamespace("SOAP-ENV");

// Tworzymy zgodne Body w przestrzeni nazw SOAP-ENV
var soapBody = XmlService.createElement("Body", soapNS);
```

Teraz nasz **dokument SOAP** (przypisany do zmiennej `soapIn`) będzie wyglądać tak:
```
<?xml version="1.0" encoding="UTF-8"?> 
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns1="https://webapi.allegro.pl/service.php">
  <SOAP-ENV:Body>
  </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

Jak widać pojawił się znacznik `Body`. Skoro mamy nasz przysłowiowy list, warto by napisać kilka miłych słów dla odbiorcy, np. z prośbą o informację zwrotną z wartością klucza wersji (niezbędny do używania WebAPI). 
W tym celu, mamy do dyspozycji metodę (doQueryAllSysStatus)[https://allegro.pl/webapi/documentation.php/show/id,62], aby się do niej odwołać, w `Body` przekażemy niezbędne dane do jej wywołania. 
W tym miejscu przyda się nam umiejętność odczytywania (dosłownie czytania, nie że parsowania czy coś) pliku **WSDL**, bo w nim zawarte są opisy metod, do których należy się odwołać. 
Chcąc odwołać się do **doQueryAllSysStatus** musimy przesłać strukturę `DoQueryAllSysStatusRequest` i dane które powinna zawierać, natomiast odbierać będziemy `doQueryAllSysStatusResponse` (to wszystko znajdziemy w WSDL - możemy go otworzyć w zwykłej przeglądarce internetowej i przy pomocy `CTRL`+`F` szukać)

```
// namespace WebApi Allegro zdefiniowany wcześniej w kopercie
var apiNS = soapEnv.getNamespace("ns1");

// Tworzymy nowy element zgodnie z dokumentacją
var methodElement = XmlService.createElement('DoQueryAllSysStatusRequest', apiNS);

// Niezbędne dane, które dołączymy w elemencie 'DoQueryAllSysStatusRequest'
var countryIdElement = XmlService.createElement('countryId', apiNS).setText(1);
var webapiKey = XmlService.createElement('webapiKey', apiNS).setText('<nasz_webapi_key>');

// Wypełniamy nasz element zdefiniowanymi danymi
methodElement.addContent(countryIdElement);
methodElement.addContent(webapiKey);

// Wypełniamy ciało naszym elementem
soapBody.addContent(methodElement);
// Pakujemy całość do koperty
soapEnv.addContent(soapBody);
```

W ten oto sposób napisaliśmy *"list"* i wsadziliśmy go do *"koperty"*, to znaczy wypełniliśmy strukturę danymi. 
Teraz zostało nam jeszcze zaadresować naszą wiadomość i ją wysłać.

## UrlFetchApp
Kolejnym serwisem Google Apps Script którego potrzebujemy, jest (UrlFetchApp)[https://developers.google.com/apps-script/reference/url-fetch/url-fetch-app], służy on do obsługi protokołu **HTTP**. 
Posłuży on do wysłania naszego *"listu"* na odpowiedni adres i odebrania odpowiedzi.

```
// Opcje dla metody fetch, definiujemy tutaj metodę (POST), typ contentu, i co wysyłamy
var options = {
	"method" : "post",
	"contentType" : "text/xml; charset=utf-8",
	"payload" : XmlService.getRawFormat().format(soapIn),
	"muteHttpExceptions" : true
};

// Teraz odwołujemy się do serwisu WebAPI na jego adres
// i dołączamy powyżej zdefiniowane opcje
var soapCall= UrlFetchApp.fetch("https://webapi.allegro.pl/service.php", options);

// Możemy wypisać co zwróciło nam API
Logger.log(soapCall);
```

## Uruchamiamy skrypt
Cały skrypt powinien wyglądać mniej więcej tak:
```
function myFunction() {
  var soapIn = XmlService.parse('<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns1="https://webapi.allegro.pl/service.php"></SOAP-ENV:Envelope>');
  // Pobieramy root node naszej koperty
  var soapEnv = soapIn.getRootElement();
  // Pobieramy jej namespace oznaczony jako 'SOAP-ENV'
  var soapNS = soapEnv.getNamespace("SOAP-ENV");

  // Tworzymy zgodne Body w przestrzeni nazw SOAP-ENV
  var soapBody = XmlService.createElement("Body", soapNS);
  // namespace WebApi Allegro zdefiniowany wcześniej w kopercie
  var apiNS = soapEnv.getNamespace("ns1");
  
  // Tworzymy nowy element zgodnie z dokumentacją
  var methodElement = XmlService.createElement('DoQueryAllSysStatusRequest', apiNS);
  
  // Niezbędne dane, które dołączymy w elemencie 'DoQueryAllSysStatusRequest'
  var countryIdElement = XmlService.createElement('countryId', apiNS).setText(1);
  var webapiKey = XmlService.createElement('webapiKey', apiNS).setText('xxxx');
  
  // Wypełniamy nasz element zdefiniowanymi danymi
  methodElement.addContent(countryIdElement);
  methodElement.addContent(webapiKey);
  
  // Wypełniamy ciało naszym elementem
  soapBody.addContent(methodElement);
  // Pakujemy całość do koperty
  soapEnv.addContent(soapBody);
  
  // Do podglądu wysyłanego requesta
  Logger.log(XmlService.getRawFormat().format(soapIn));
  
  // Opcje dla metody fetch, definiujemy tutaj metodę ('POST'), typ contentu, i co wysyłamy
  var options = {
    "method" : "post",
    "contentType" : "text/xml; charset=utf-8",
    "payload" : XmlService.getRawFormat().format(soapIn),
    "muteHttpExceptions" : true
  };
  
  // Teraz odwołujemy się do serwisu WebAPI na jego adres
  // i dołączamy powyżej zdefiniowane opcje
  var soapCall= UrlFetchApp.fetch("https://webapi.allegro.pl/service.php", options);
  
  // Możemy wypisać co zwróciło nam API
  Logger.log(soapCall);
}
```

Teraz wystarczy go uruchomić, udzielić uprawnień i sprawdzić **Dziennik (Log)**. 
<figure class="center">
	<img src='{{ site.url }}/images/gas/gas_soap_run.gif' alt="">
	<figcaption>Uruchomienie skryptu</figcaption>
</figure>

Poprawnie zwrócony response powinien wyglądać tak:
```
<?xml version="1.0" encoding="UTF-8"?>
<SOAP-ENV:Envelope xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/" xmlns:ns1="https://webapi.allegro.pl/service.php">
  <SOAP-ENV:Body>
    <ns1:doQueryAllSysStatusResponse>
      <ns1:sysCountryStatus>
        <ns1:item>
          <ns1:countryId>1</ns1:countryId>
          <ns1:programVersion>1.0</ns1:programVersion>
          <ns1:catsVersion>1.8.1</ns1:catsVersion>
          <ns1:apiVersion>1.0</ns1:apiVersion>
          <ns1:attribVersion>1.0</ns1:attribVersion>
          <ns1:formSellVersion>1.17.25</ns1:formSellVersion>
          <ns1:siteVersion>1.0</ns1:siteVersion>
          <ns1:verKey>1531858541</ns1:verKey>
        </ns1:item>
      </ns1:sysCountryStatus>
    </ns1:doQueryAllSysStatusResponse>
  </SOAP-ENV:Body>
</SOAP-ENV:Envelope>
```

# Skrypt z Logowaniem
Umieszam kod bardziej rozbudowanej wersji skryptu, którą można realnie wykorzystać do integracji WebApi Allegro przez SOAP z Google Apps Script. 
Posiada możliwość logowania się (uwierzytelniania sesji), oraz bardziej rozbudowane mechanizmy parsowania XML. 
Niestety, jest to tylko **proof-of-concept** z powodu problemów, które są nie do przeskoczenia, ale można się pobawić.

GitHub: (gas-allegrowebapi)[https://github.com/chawel/gas-allegrowebapi]

# Podsumowanie
Cały skrypt jak już wspomniałem, to tylko *"pokazówka"* możliwości GAS i chyba niezbyt trafnie wybrałem WebAPI Allegro na ten przykład. Dlaczego? Z dwóch powodów:
1. Allegro zapowiedziało rozpoczęcie procesu *"wygaszania"* WebAPI i przeniesienia całej funkcjonalności do (REST API)[https://developer.allegro.pl/news/2018-06-06-Wygaszamy_webapi/]
2. WebAPI posiada dość osobliwe zabezpieczenie, IP z którego uzyskano `sessionId` musi być stałe jeżeli chodzi o dalsze używanie zasobów API. Niestety Google nie oferuje statycznego adresu IP dla swoich usług, więc często już podczas uwierzytelnienia, nasz `sessionId` jest bezużyteczny, ponieważ Google przeniosło nas na inny serwer (a co za tym idzie wysyłamy requesty z nowego adresu IP). Więcej o tym zabezpieczeniu: (https://allegro.pl/webapi/faq.php#faq_5)[https://allegro.pl/webapi/faq.php#faq_5]

Niemniej jednak, blog prowadzę z myślą o osobach zajmujących się sprzedażą internetową, a pierwsze API obługującym SOAP które przyszło mi do głowy było właśnie WebAPI Allegro. Opisane tutaj metody pracy GAS z SOAP i formatem XML są uniwersalne, więc swobodnie można na podstawie tej instrukcji zbudować integrację z innym WS. Już niedługo - kolejna część a w niej opis jak połączyć REST API z Google SpreadSheets!
