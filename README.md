# Clash Royale — pipeline

## Pliki

**`CR_CRAWLER_INGESTOR`** (AWS Lambda) pobiera historię bitew graczy z kolejki w DynamoDB przez API Clash Royale, odrzuca bitwy spoza trybów rankingowych i zakresu 7000–11 000 pucharów, usuwa duplikaty i zapisuje wynik do `raw/`. Przy okazji dodaje część przeciwników do kolejki, a gdy kolejka jest pusta, zasila ją graczami z globalnego rankingu.

**`CR_Data_Compactor`** (AWS Lambda) scala małe pliki z `raw/` w jedną paczkę liczącą co najmniej 10 000 bitew, zapisuje ją do `final_raw/` i usuwa pliki źródłowe.

**`CR_Data_Transformer`** (AWS Lambda, wyzwalana plikiem w `final_raw/`) czyści dane, normalizuje poziomy kart i wież do wspólnej skali, zamienia układ p1/p2 na zwycięzca/przegrany i wylicza cechy talii, takie jak średni koszt eliksiru, poziomy kart i liczby kart według rzadkości. Wynik trafia do `silver/`.

**`cr-standardize-archive`** (AWS Glue) przekształca archiwalny zbiór z Kaggle (2020) do schematu zgodnego z danymi bieżącymi: zamienia identyfikatory kart na nazwy, ujednolica typy i uzupełnia zerami kolumny mechanik, których w 2020 roku jeszcze nie było.

**`cr-standardize-current`** (AWS Glue) zamienia nazwy kolumn w `silver/` na `snake_case` i usuwa kolumny, których nie ma w danych archiwalnych.

## Struktura bucketu S3

```
s3-bucket/
├── raw/                          ← CR_CRAWLER_INGESTOR (małe pliki z pojedynczych uruchomień)
├── final_raw/                    ← CR_Data_Compactor (paczki po ≥10 000 bitew)
├── silver/                       ← CR_Data_Transformer, potem cr-standardize-current
├── clash-royale-archive-2020/    ← zbiór z Kaggle, przekształcany przez cr-standardize-archive
└── metadata/
    └── card_stats.json           koszty, rzadkości i identyfikatory kart
```

