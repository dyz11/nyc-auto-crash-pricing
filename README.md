# NYC Auto Crash Risk: A Pricing-Style Frequency Study

![NYC crash map](nyc_crash_map.png)

*Police-reported crashes, 2023–2025. Orange = no borough recorded (mostly highways).*

## Question

How much does car crash risk differ across NYC's five boroughs and across times of day, once the number of vehicles per borough is accounted for, and how close can public data get to the rating an insurer would use?

## Data & exposure

I used NYC Open Data's police-reported crash records for 2023–2025 (273,469 crashes) and NYS DMV vehicle registrations (2,140,181 vehicles registered in the five boroughs with registrations expiring after 1/1/2026, excluding trailers) as exposure. About 27% of crashes had no borough, mostly on highways plus a few neighborhoods where borough wasn't recorded. Rather than dropping them, I allocated them to boroughs using the percentage of registered cars attributed for each borough as an assumption, and compared that against dropping them as a sensitivity check. I modeled crash counts by borough, time of day, weekday/weekend and year with a Negative Binomial GLM, using registered vehicle-days (time when an accident is possible) as exposure.

## Findings

- Brooklyn vehicles have about 1.6× the crash rate of Queens (1.6–1.9× depending on how highway crashes are assigned); the Bronx about 1.5×; Staten Island about 0.6×.
- The evening rush (4–8pm) is the riskiest time, about 1.15× midday. Night has 41% fewer crashes per hour, but night crashes are about 3× as likely to be fatal (5.5 vs. 1.8 deaths per 1,000 crashes).
- Crash counts varied about 70× more than Poisson allows, so a Negative Binomial model was needed for honest confidence intervals.

## Limitations

- **Crash location vs. garaging location.** Insurers rate by where a car is kept, however this data records where crashes happen. This inflates Manhattan (1.9×, treated as an upper bound) because of higher activity, and it likely understates Queens in the same way.
- **No traffic volume:** time-of-day results reflect how many cars are on the road, not risk per mile.
- **Police-reported crashes only:** there could be a significant number of minor accidents that are not reported.
- **No claim dollars:** this measures how often crashes happen, not what they cost.

## What I'd do with insurer data

Use real exposure (car-years and miles driven), assign claims to the car's garaging address, model claim severity in dollars alongside frequency, add considerations for driver characteristics, and use credibility weighting so small groups lean on broader averages.

## Data

All data is free and public. The raw files are not included in this repository because of their size; follow the steps below to download them.

### 1. Crash records: NYC Open Data
**Source:** [Motor Vehicle Collisions – Crashes](https://data.cityofnewyork.us/Public-Safety/Motor-Vehicle-Collisions-Crashes/h9gi-nx95) (NYPD, dataset `h9gi-nx95`)

Every police-reported motor vehicle crash in NYC, one row per crash.

**How to download:**
1. Open the dataset page and click **Export → CSV**.
2. Filter **Crash Date** to 01/01/2021 – 12/31/2025. Apply **no other filters**: keep all boroughs, including crashes with a blank borough.
3. Save the file as `Motor_Vehicle_Collisions_-_Crashes_20260926.csv` in the same folder as the notebook.

The export used here has 487,914 crashes (2021–2025). The analysis uses **2023–2025: 273,469 crashes**.

**Columns used:** crash date and time, borough, latitude/longitude, on-street name, number of persons injured and killed, contributing factor (vehicle 1), vehicle type (vehicle 1), collision ID.

### 2. Exposure: NYS DMV vehicle registrations
**Source:** [Vehicle, Snowmobile, and Boat Registrations](https://data.ny.gov/Transportation/Vehicle-Snowmobile-and-Boat-Registrations/w4pv-hbkt) (NYS DMV, dataset `w4pv-hbkt`)

Used as exposure: the number of registered vehicles in each borough.

**Filters applied** 
- **Record Type** = VEH (vehicles only; excludes boats and snowmobiles)
- **County** = Kings, Queens, New York, Bronx, Richmond (the five boroughs)
- **Reg Expiration Date** after 01/01/2026
- **Body Type** is not TRLR (excludes trailers, which can't crash on their own)

Then **Group and aggregate** by County, counting VIN. These counts are entered directly in the notebook, so this file does not need to be downloaded to run it.

| Borough | DMV county | Registered vehicles |
|---|---|---|
| Queens | Queens | 819,953 |
| Brooklyn | Kings | 530,238 |
| Staten Island | Richmond | 290,205 |
| Bronx | Bronx | 266,992 |
| Manhattan | New York | 232,793 |
| **Total** | | **2,140,181** |

### Data preparation notes
- **Missing borough:** 27% of 2023–2025 crashes have no borough recorded, mostly highway crashes plus a few neighborhoods where borough wasn't recorded (see map). Instead of dropping them, they were allocated to boroughs in proportion to registered vehicles; dropping them is shown as a sensitivity check.
- **Map:** crashes without usable coordinates (about 7%) are excluded from the map only.
- **Registration snapshot:** the DMV file is a current snapshot, not a historical one. Including trailers (2,168,454 vehicles) or using a 09/26/2026 snapshot (2,110,255) changes relativities by less than 0.02.
- **Insurance price comparison:** average full-coverage premiums by area from [Insurify](https://insurify.com/car-insurance/new-york/average-cost/) (updated August 2026), used only to compare with the model's results.

## How to run

1. Download the crash CSV as described above and put it in the same folder as the notebooks.
2. Install the libraries: `pip install pandas numpy statsmodels matplotlib requests jupyter`
3. Open `nyc_auto_crash_risk.ipynb` (main analysis) or `nyc_crash_map.ipynb` (map) and run all cells.
