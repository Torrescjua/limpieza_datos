# limpieza\_datos

A modular pipeline for cleaning and harmonising public‐sector datasets in Colombia. The project ingests raw files from three strategic domains—**education**, **telecommunications** and **RedVolución**—and produces standardised, analysis‑ready tables at municipal resolution.

---

## 1  Project goals

* **Data consistency**   Convert heterogeneous CSV/XLS sources into a single, tidy schema.
* **Geographic alignment**   Key all records to official DANE department/municipality codes.
* **Quality assurance**   Detect and correct nulls, range violations and time‑series gaps.
* **Cross‑domain insight**   Enable integrated policy analytics across sectors.

---

## 2  System architecture

```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│  Education   │      │   Telecom    │      │  RedVolución │
└──────┬───────┘      └──────┬───────┘      └──────┬───────┘
       ▼                    ▼                    ▼
      Raw                  Raw                  Raw
   (ICFES CSV)        (ISP reports)        (Project CSV)
       │                    │                    │
       ├── cleanse & QC ────┼── cleanse & QC ────┼── cleanse & QC
       │                    │                    │
       ▼                    ▼                    ▼
   Resultados_          Internet_Fijo_        RedVolucion_
   icfes_limpios.zip    Penetracion_          Limpio.csv
                        Limpio.csv
       │                    │                    │
       └──────────┬─────────┴─────────┬──────────┘
                  ▼                   ▼
             Cross‑domain joins   Analytical reports
```

Each domain owns its *cleansing sub‑pipeline* but shares the same validation framework and naming conventions.

---

## 3  Processed domains

| Domain          | Raw source                                  | Clean output                                              | Key transformations                                               |
| --------------- | ------------------------------------------- | --------------------------------------------------------- | ----------------------------------------------------------------- |
| **Education**   | National ICFES exam results                 | `Resultados_icfes_limpios.zip`                            | Score parsing, institution metadata, student demographics         |
| **Telecom**     | Fixed‑internet penetration reports (MinTIC) | `Internet_Fijo_Penetracion_Municipio_YYYYMMDD_Limpio.csv` | Access‑rate normalisation, population join, rate calculation      |
| **RedVolución** | Internal project management CSV             | `RedVolucion_YYYYMMDD_Limpio.csv`                         | Investment per capita, temporal normalisation, indicator encoding |

---

## 4  Data model (example: RedVolución)

| Column                            | Description                                  |
| --------------------------------- | -------------------------------------------- |
| `DEPARTAMENTO`                    | Department name (uppercase, ASCII)           |
| `MUNICIPIO`                       | Municipality name                            |
| `COD_MUNICIPIO`                   | Unique DANE municipality code                |
| `FECHA_CARGUE` / `FECHA_VIGENCIA` | Load and validity dates                      |
| `INVERSION_TOTAL`                 | Total investment (COP)                       |
| `INVERSION_POR_PERSONA`           | Normalised metric used in cross‑domain joins |

*(Education and Telecom follow an analogous, normalised schema.)*

---

## 5  Naming conventions

```
{Dominio}_{yyyymmdd}_Limpio.{ext}
```

* `Limpio`  ⇢ passed all QC checks
* `yyyymmdd`  ⇢ batch timestamp (UTC‑5)
* Extensions  ⇢ `.csv` for single‑sheet files, `.zip` for multi‑sheet bundles

---

## 6  Geographic integration framework

All tables include the fields `COD_DEPARTAMENTO` and `COD_MUNICIPIO`, enabling fast joins at municipal granularity. These keys unlock:

* **horizontal joins** across education ↔︎ telecom ↔︎ project datasets
* **choropleth maps** and spatial statistics
* **hierarchical roll‑ups** (municipality → department → national)

---

## 7  Data‑quality rules

| Check category      | Rule                                                  |
| ------------------- | ----------------------------------------------------- |
| **Completeness**    | Mandatory fields not null; population coverage ≥ 95 % |
| **Temporal**        | No missing quarters in 2015‑2018 window               |
| **Geographic**      | DANE codes valid and consistent with names            |
| **Domain‑specific** | Score ranges, investment ≥ 0, penetration 0‑1         |

The validation layer logs any violations and produces a rejection report for manual follow‑up.

---

## 8  Integration & analytics

* **Domain‑specific pipelines** produce curated tables for each sector.
* **Cross‑domain workflows** combine tables by municipality and year to study relationships such as:

  * Internet penetration vs. student performance
  * Investment per capita vs. digital literacy gains
* **Temporal alignment** is guaranteed by synchronised batch timestamps (`yyyymmdd`).

---

## 9  Usage

```bash
# Clone repo and install deps
pip install -r requirements.txt

# Run all pipelines for current day
python main.py --run-date $(date +%Y%m%d)

# Output files will appear in ./output/{dominio}/
```

---

## 10  Notes

* The project is tailored to Colombian administrative geography and leverages official DANE codes.
* All processing is Spark‑based; adjust `spark.executor.memory` in `config.yml` for larger batches.
* Contributions and issue reports are welcome—please open a PR or GitHub issue.
