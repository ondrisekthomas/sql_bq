# SSOT Query Builder

Lokální no-code generátor BigQuery SQL pro deset SSOT zdrojů a statická metadata z `datahub-core-prod.edit_perf_v2.articles`. Neodesílá žádná data a nevyžaduje instalaci balíčků.

```bash
python3 -m http.server 8000
```

Potom otevřete `http://localhost:8000`.

Katalog metrik a dimenzí je v `app.js`. Rozšířené položky se vybírají přes vyhledávání s našeptávačem; nejpoužívanější volby zůstávají na jedno kliknutí. Generátor vždy přidává datumový filtr, každý zdroj nejdřív agreguje do vlastního CTE a až potom zdroje spojuje. Tím omezuje násobení metrik při joinu. U statických článků používá datum publikace a aliasuje `documentId` na společný `document_id`.

Složitá pole mají v katalogu typová metadata a vlastní SQL transformaci. Například `rubrics` převádí JSON pole názvů rubrik na text oddělený ` | `, `rubric_title` vybere první rubriku a `authors` rozbalí JSON pole uživatelů. Další JSON/ARRAY/RECORD pole se přidávají stejným způsobem přes `dimensionMeta`, `dimensionSql` a `filterSql`.

Zdroje jsou rozdělené do záložek SSOT, Articles a Social. Social obsahuje používané tabulky z datasetu `datahub-reporting.aggregate_facebook`. Kumulativní snapshoty se nejprve deduplikují na jeden záznam za entitu a den, pomocí `LAG` a vzoru `IFNULL(COALESCE(current - LAG(current), current), 0)` se převedou na denní přírůstky a teprve ty se sčítají. Generátor načítá 35denní lookback, aby byl správný i první den zvoleného období. Explicitně inkrementální zdroje používají přímo `SUM`. Zastaralé metriky s `impressions` nejsou v nabídce.
