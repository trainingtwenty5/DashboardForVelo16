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

## Statusy

| Status | Znaczenie |
|---|---|
| **Dostarczony** | mail wyszedł i nie wróciło odbicie |
| **Odbity (nieaktualny)** | skrzynka odbiorcy odrzuciła wiadomość — najczęściej adres już nie istnieje |
| **Błąd SMTP** | nasz serwer nie przyjął wysyłki (np. kod 534 — Gmail odrzucił hasło aplikacji); **mail w ogóle nie wyszedł**, warto wysłać ponownie |
| **W kolejce** | adres jest na liście kontaktów, ale jeszcze nie było próby wysyłki |

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
