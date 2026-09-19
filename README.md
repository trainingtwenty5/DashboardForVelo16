# Dashboard Velo16

Statyczna strona z podsumowaniem wysyłki maili Velo16 — jeden plik `index.html`,
bez backendu, bez zależności. Generuje ją skrypt `velo16_monitor.py`
(folder `Real_estate_e/Velo_emieler` na komputerze).

## Co pokazuje

Zakładki odpowiadają folderom kampanii:

| Zakładka | Folder na dysku |
|---|---|
| Rowery Polska | `Velo_emieler/` (główny) |
| Buty | `Velo_emieler/Buty/` |
| Francja — narty | `Velo_emieler/Francja/` |
| E-bike góry polskie | `Velo_emieler/Sewisy_i_wypozyczalnie_rowerow_elektrycznych_w_gorach_polskich/` |

Nowy folder z logami wysyłki jest wykrywany automatycznie i dostaje własną zakładkę.

Dla każdej kampanii: liczby zbiorcze, wykres dzień po dniu, wykres narastający,
podział całej bazy i przeszukiwalne listy adresów z pobieraniem do CSV.

## Sprzedaż liczona wątkami

Jedna sprzedaż to **cały wątek**: oferta → odpowiedź → zamówienie → faktura.
Wchodzi do sumy **raz**. Wątek sklejany jest po trzech śladach:

1. nagłówki `References` / `In-Reply-To` — to jest pewne,
2. ten sam temat (bez `RE:`) od tego samego kontrahenta,
3. ta sama kwota od tego samego kontrahenta w ciągu 45 dni — tak łączy się
   zamówienie z fakturą, która go nie cytuje.

O kwocie decyduje najmocniejszy dokument w wątku: **faktura > zamówienie >
kwota z treści maila**. Gdy faktura jest skanem bez pozycji, kwota zostaje
z faktury, a lista pozycji („co sprzedaliśmy") z zamówienia w tym samym wątku.

Zakładka **Faktury i kwoty** pokazuje kolejno: kafelki sprzedaży, podział na
kampanie, tabelę transakcji wątek po wątku, listę sprzedanego towaru,
wykresy per waluta i na końcu **wszystkie maile z kwotą** jako materiał
źródłowy — tam jeden wątek może mieć kilka wierszy.

## Faktury, kwoty i prowizja

Zakładka **Faktury i kwoty** zbiera każdą kwotę wykrytą w skrzynce — z treści
maila albo z załącznika.

* **Prowizja 0,5% stoi przy każdej kategorii.** Ma ją każda kampania w tabeli
  „Podział na kampanie" (także ta, w której nie było jeszcze żadnej faktury —
  wtedy `0,00`), każda zakładka kampanii w swoich kafelkach, wiersz `Razem`,
  każdy produkt w tabeli „Co się sprzedało" i każda kolumna w CSV.
* **Osobno dla każdej waluty.** PLN i EUR mają własne kafelki, własne wiersze
  w tabeli i własny wykres — mieszanie ich na jednej osi dawało słupki, których
  nie da się porównać.
* **Do sumy i prowizji wchodzą tylko pewne odczyty**, czyli kwoty stojące przy
  etykiecie w rodzaju „razem do zapłaty". Zgadnięte są na liście, ale poza sumą.

### Skąd brane są liczby z faktury

| Źródło | Co się z niego czyta |
|---|---|
| **XML KSeF (FA 1/2/3)** | numer, data, waluta, suma `P_15` i pozycje `FaWiersz` — wprost z dokumentu, bez zgadywania |
| **XML UBL / PEPPOL** | `PayableAmount`, numer, data i pozycje faktury |
| **PDF** | tekst **i tabele** (pdfplumber) — wiersz tabeli trafia do odczytu jako jedna linia |
| **HTML** | tabela zamieniana na wiersze i kolumny, encje (`&nbsp;`, `&#347;`) rozkodowane |
| **XLSX / DOCX / CSV** | tekst z zachowaniem podziału na kolumny |

Pliki, które sami wysyłamy w kampanii (listy kontaktów, oferty, cenniki), są
pomijane przy liczeniu kwot — **chyba że wrócą jako zamówienie**.

### Zamówienia w arkuszu

Kontrahent zwykle odsyła nasz własny plik oferty z dopisaną kolumną
`zamówienie`. Skrypt szuka w arkuszu nagłówka z kolumną nazwy, ilości
zamawianej i ceny, po czym liczy **ilość × cena** po wszystkich wierszach.
Wartością dokumentu jest suma zamówienia, a nie pierwsza kwota znaleziona
w mailu — wcześniej oferta „15 zł brutto/para" cytowana w odpowiedzi dawała
zamówienie na 15 zł.

Zabezpieczenia: komórki w rodzaju `10+` (stan magazynowy) nie są ilością,
a w **naszym** pliku kolumna musi wprost mówić o zamówieniu — samo „ilość"
w cenniku to stan, nie zamówienie.

## Statusy

| Status | Znaczenie |
|---|---|
| **Dostarczony** | mail wyszedł i nie wróciło odbicie |
| **Odbity (nieaktualny)** | skrzynka odbiorcy odrzuciła wiadomość — najczęściej adres już nie istnieje |
| **Błąd SMTP** | nasz serwer nie przyjął wysyłki (np. kod 534 — Gmail odrzucił hasło aplikacji); **mail w ogóle nie wyszedł**, warto wysłać ponownie |
| **W kolejce** | adres jest na liście kontaktów, ale jeszcze nie było próby wysyłki |

## Błąd SMTP — co dalej

Kafelek **Błąd SMTP** liczy adresy, do których mail nie wyszedł w ogóle.
Baner pod nagłówkiem pokazuje, **z jakiego powodu** (zerwane połączenie,
dzienny limit Gmaila 550, żądanie logowania 534, serwer zajęty 421) i ile
z tych adresów można ponowić od ręki.

Mailer dopisuje adres do `*_sent_history.json` dopiero **po udanej wysyłce**,
więc adres z samym błędem nie jest zablokowany — wróci do kolejki sam. Dlatego
status nazywa się **„Do ponowienia"**, a kafelek **„w kolejce razem"** liczy
adresy bez próby plus te do ponowienia.

Sprawdzić to można skryptem `velo16_ponow_bledy.py`: pokazuje, ile adresów
nie doszło, ile z nich siedzi w historii (powinno być 0) i czy wszystkie są
nadal na liście kontaktów, z której czyta mailer. Z `--zrob` usuwa z historii
i zostawia kopię pliku.

## Skąd biorą się dane

* `*_wysylka_log_*.txt` — linie `WYSŁANO` i `BŁĄD`, liczone od pierwszego dnia logów
* `*_sent_history.json` — historia wysyłek
* `*.xlsx` — listy kontaktów (kto jeszcze czeka w kolejce)
* IMAP skrzynek Velo16 — odbicia (z adresem, który odbił) i odpowiedzi

## Prywatność

W tym repozytorium adresy e-mail są **zamaskowane** (`k*****t@firma.pl`).
Pełna wersja zostaje na dysku, w `Velo_emieler/index.html`.

## Odświeżenie

```
python velo16_monitor.py --once
```

potem `wypchnij_dashboard.bat` z folderu `Velo_emieler`.
