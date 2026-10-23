# Raport — Laboratorium 03

**Zespół:**  1

**Data:**  22.10.2026

## Środowisko

| Parametr | Wartość |
|----------|---------|
| DeepSeek-V3.1 | |
| Liczba dostępnych notatek | 10 |

(Plik labeled-notes.csv nie ma wypełnionych kolumn floor i elevator potrzebnych do sprawdzenia poprawnosci działania modelu, 
dlatego eksperyment był oparty tylko o informacje z pliku examples.csv)

## EXP-00 — Sposób oceny wyników

Jak oceniamy poprawność odpowiedzi modelu? Jakie metryki przyjmujemy i dlaczego?

Poprawność odpowiedzi modelu oceniamy poprzez porównanie
wyekstraktowanych przez niego danych z wzorcem
zawartym w pliku z danymi.


| Metryka | Definicja |
|---------|-----------|
| Floor accuracy|	Odsetek poprawnie rozpoznanych pięter|
|Elevator accuracy|	Odsetek poprawnie rozpoznanych informacji o windzie|
|Exact match|	Odsetek notatek, dla których oba pola są poprawne| 

**Dlaczego te metryki?**

Floor accuracy - Pozwala ocenić, jak dobrze model radzi sobie z wyciąganiem danych
liczbowych i określaniem parteru (0) lub piwnic (-1).

Elevator accuracy - Weryfikuje zdolność modelowania niuansów 
logicznych i detekcji obecności infrastruktury w tekście.

Exact match - Pokazuje, w ilu przypadkach model był całkowicie poprawny.

## EXP-01 — Porównanie modeli 

Dla wariantu z tylko modelem DeepSeek-V3.1, będzie to sprawdzenie wyników.

| Model | Floor accuracy | Elevator accuracy | Exact match | Średni czas [ms] |
|-------|----------------|-------------------|-------------|------------------|
| DeepSeek-V3.1| 1 | 0.8| 0.8| 376|


**Exact match** oznacza odsetek notatek, dla których model poprawnie rozpoznał jednocześnie oba pola: `floor` i `elevator`.

**Wnioski:**
Model DeepSeek-V3.1 wykazuje bardzo wysoką skuteczność ekstrakcji dla pięter (100%) oraz dość wysoką skuteczność ekstrakcji dla wind (80%).
Ponadto średni czas czas odpowiedzi jest bardzo niski (376 ms).


## EXP-02 — Prompting

| Wariant | Floor accuracy | Elevator accuracy | Exact match |
|---------|----------------|-------------------|-------------|
| zero-shot |1	|0.8	|0.8	|
| one-shot |1	|0.8	|0.8	|
| few-shot |1	|0.8	|0.8	|


**Wnioski:**
Wszystkie trzy warianty (zero-shot, one-shot, few-shot) osiągnęły identyczne wyniki. Wskazuje to, że DeepSeek-V3.1 dysponuje na tyle silną bazową znajomością języka naturalnego oraz umiejętnością trzymania się instrukcji, że dodawanie przykładów w prompcie nie wnosiło już żadnej dodatkowej wartości dla tak prostego zadania ekstrakcji.

## EXP-03 — Temperature

| Temperature | Exact match | Powtarzalne notatki |
|-------------|-------------|---------------------|
| 0.0 | 0.8| |
| 0.5 | 0.78| |
| 1.0 |0.72 | |



**Wnioski:**
Wraz ze wzrostem temperatury dokładność i powtarzalność modelu spadała.Wyższa temperatura wprowadza element losowy, co prowadzi do częstszych błędów.

## EXP-04 — Structured output — JSON i walidacja

**json-mode** - natywny tryb API

| Wariant | Poprawny JSON | Exact match |
|---------|---------------|-------------|
| prompt | 10| 0.8|
| json-mode | 6| 0.4|

**Wnioski:**
Wariant oparty na czystym prompcie (prompt) okazał się znacznie bardziej stabilny 
i skuteczny niż natywny tryb API (json-mode).

Instrukcja zawarta w prompcie zagwarantowała 100% poprawności strukturalnej JSON-a (10/10).

Natywny tryb API (json-mode) w tym konkretnym połączeniu z endpointem powodował błędy składniowe i zwracał niepoprawne struktury z pustymi kluczami w 4 przypadkach na 10.
Na przykład:

✗ NOTE-002 "2p bez windy, drzwi na końcu" => Niepoprawny JSON: {
{
  "": 2
}}

Zatem, że przy pracy z zewnętrznymi lub akademickimi endpointami LLM, dobrze zaprojektowana instrukcja w treści promptu bywa bardziej niezawodna niż automatyczne wymuszanie formatu na poziomie parametrów API.

## Najtrudniejsze notatki

Które notatki model najczęściej interpretował błędnie i dlaczego?

Największe problemy model napotkał przy notatkach:

- ✗ NOTE-007 "4 piętro, winda tylko do 3" => floor=4, elevator=false (oczekiwano: floor=4, elevator=true)

- ✗ NOTE-010 "Winda nie działa, dostawa na 5p" => floor=5, elevator=false (oczekiwano: floor=5, elevator=true)

Notatki te zawierają negację i pewną warunkowość dotyczącą windy. Model nadmiernie 
upraszczał informację dotyczące windy i zinterpretował je jako brak tego elementu.


