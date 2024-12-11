# wrc-db-utils
Utilities for grabbing and working with WRC timing data in a SQLite database

View in datasettelite using eg https://raw.githubusercontent.com/RallyDataJunkie/wrc-db-utils/main/wrc_2024_archive_results.db?raw=true

For example: https://lite.datasette.io/?url=https%3A%2F%2Fraw.githubusercontent.com%2FRallyDataJunkie%2Fwrc-db-utils%2Fmain%2Fwrc_2024_archive_results.db%3Fraw%3Dtrue

---

Split times as seconds ([example](https://lite.datasette.io/?url=https%3A%2F%2Fraw.githubusercontent.com%2FRallyDataJunkie%2Fwrc-db-utils%2Fmain%2Fwrc_2023_archive_results.db%3Fraw%3Dtrue#/wrc_2023_archive_results?sql=SELECT+%0A++++stageId%2C+carNo%2C%0A++++round1%2C+%0A++++CASE+%0A++++++++WHEN+ROW_NUMBER%28%29+OVER+%28PARTITION+BY+stageId+ORDER+BY+pos%29+%3D+1+THEN+%0A++++++++++++ROUND%28CAST%28SUBSTR%28round1%2C+1%2C+INSTR%28round1%2C+%27%3A%27%29+-+1%29+AS+FLOAT%29+*+60+%2B+CAST%28SUBSTR%28round1%2C+INSTR%28round1%2C+%27%3A%27%29+%2B+1%29+AS+FLOAT%29%2C+1%29%0A++++++++ELSE+%0A++++++++++++ROUND%28%0A++++++++++++++++FIRST_VALUE%28CAST%28SUBSTR%28round1%2C+1%2C+INSTR%28round1%2C+%27%3A%27%29+-+1%29+AS+FLOAT%29+*+60+%2B+CAST%28SUBSTR%28round1%2C+INSTR%28round1%2C+%27%3A%27%29+%2B+1%29+AS+FLOAT%29%29+OVER+%28PARTITION+BY+stageId+ORDER+BY+pos%29+%0A++++++++++++++++%2B+CAST%28REPLACE%28round1%2C+%27%2C%27%2C+%27.%27%29+AS+FLOAT%29%2C+%0A++++++++++++1%29+%0A++++END+AS+sectortime_s+%0AFROM+splittimes)):

```sql
SELECT 
    stageId, carNo,
    round1, 
    CASE 
        WHEN ROW_NUMBER() OVER (PARTITION BY stageId ORDER BY pos) = 1 THEN 
            ROUND(CAST(SUBSTR(round1, 1, INSTR(round1, ':') - 1) AS FLOAT) * 60 + CAST(SUBSTR(round1, INSTR(round1, ':') + 1) AS FLOAT), 1)
        ELSE 
            ROUND(
                FIRST_VALUE(CAST(SUBSTR(round1, 1, INSTR(round1, ':') - 1) AS FLOAT) * 60 + CAST(SUBSTR(round1, INSTR(round1, ':') + 1) AS FLOAT)) OVER (PARTITION BY stageId ORDER BY pos) 
                + CAST(REPLACE(round1, ',', '.') AS FLOAT), 
            1) 
    END AS sectortime_s 
FROM splittimes
```
