# Tourism Tour Packages (Moscow → Europe)

## Project Description

A data collection, integration, and analysis project for building tourist tour packages across European destinations.

**Business goal:** help travelers find optimal tours (flight + hotel) based on various criteria — price, rating, location, and value for money.

**Search parameters:**
- Departure from Moscow (MOW)
- Dates: July 1 — July 15, 2026 (14 nights)
- 2 adults, 1 room
- 25 European cities

**Target variable:** `total_price` = flight price + hotel price (EUR)

## Data Sources

| Source | Collection Method | Volume | Description |
|--------|-------------------|--------|-------------|
| [Travelpayouts API](https://www.travelpayouts.com/) | REST API (5 endpoints) | ~10,700 records (reference data) + 235 flights | Flight prices, city and airport directories, popular destinations |
| [Booking.com](https://www.booking.com/) | Selenium (web scraping) | 19,166 hotels | Hotels: name, price, rating, distance to center, reviews |

## Project Structure

```
apiavia/
├── project.ipynb                 # Main notebook (full pipeline)
|
├── .gitignore
├── README.md
└── data/
    ├── raw/                      # Raw data
    │   ├── api_cities.csv
    │   ├── api_airports.csv
    │   ├── api_popular_european_destinations.csv
    │   ├── api_prices_for_dates.csv
    │   ├── api_grouped_prices.csv
    │   └── booking_hotels_all_cities.csv
    └── clean/                    # Cleaned data
        ├── clean_flights.csv
        ├── clean_hotels.csv
        └── clean_grouped_prices.csv
```

## Pipeline

The entire pipeline is implemented in **`project.ipynb`** and consists of 4 parts:

1. **Flight data collection (API)** — 5 requests to the Travelpayouts API: city and airport directories, popular European destinations, prices for specific dates, grouped minimum prices
2. **Hotel scraping (Selenium)** — data collection from Booking.com: 25 cities × 16 pages × ~25 hotels per page. Extracted fields: name, price, rating, distance to center, review count
3. **Data cleaning** — parsing text fields into numeric values, removing duplicates, handling missing values
4. **Tour recommendations** — merging flights and hotels (inner join on city), building rankings: cheapest tours, best value for money, city comparison

---

## Data Description

### 1. Flights — `api_prices_for_dates.csv`

Flight prices from Moscow to European cities. **235 records, 17 features, 0 missing values.**

| Feature | Type | Description | Example |
|---------|------|-------------|---------|
| `flight_number` | int | Flight number | 2170 |
| `link` | str | Booking link | `/search/MOW2407LON1?...` |
| `origin_airport` | str (IATA) | Departure airport | `SVO`, `VKO`, `DME` |
| `destination_airport` | str (IATA) | Arrival airport | `LGW`, `FCO`, `BCN` |
| `departure_at` | str  | Departure date and time | `2026-07-24T01:00:00+03:00` |
| `airline` | str  | Airline code | `SU`, `UT`, `DP` |
| `destination` | str  | Destination city code | `LON`, `ROM`, `MIL` |
| `return_at` | str  | Return date | `2026-07-15T...` |
| `origin` | str  | Origin city code | `MOW` (always) |
| `price` | int | Ticket price, EUR | 203–1234 |
| `gate` | str | Booking agency | `Kupi.com`, `Aviakassa` |
| `return_transfers` | int | Return flight transfers | 0–2 |
| `duration` | int | Total duration, min | 580–1935 |
| `duration_to` | int | Outbound duration, min | 355–760 |
| `duration_back` | int | Return duration, min | 0–850 |
| `transfers` | int | Outbound transfers | 0–3 |
| `requested_destination` | str | Requested city (code) | `LON`, `ROM` |

**Destinations:** 9 unique cities. **Airlines:** 21 (SU, UT, DP, U6, S7, etc.).

### 2. Grouped Prices — `api_grouped_prices.csv`

Minimum prices grouped by departure date. **233 records, 18 features, 0 missing values.** Same structure as flights + `group_key` (str) — departure date as the grouping key.

### 3. Popular Destinations — `api_popular_european_destinations.csv`

Cheapest flights to Europe from Moscow, filtered by ISO codes of European countries. **9 records, 17 features, 0 missing values.** Additional fields: `destination_name` (str — `Milan`, `London`), `destination_code` (str — `MIL`, `LON`), `country_code` (str — `IT`, `GB`).

### 4. City Directory — `api_cities.csv`

Full Travelpayouts city reference. **9,641 records, 8 features.**

| Feature | Type | Description | Missing |
|---------|------|-------------|---------|
| `name_translations` | str (JSON) | Names in multiple languages | 0 |
| `cases` | str (JSON) | Grammatical cases (Russian) | 0 |
| `country_code` | str (ISO) | Country code | 31 |
| `code` | str (IATA) | IATA city code | 0 |
| `time_zone` | str | Time zone | 0 |
| `name` | str | City name (English) | 0 |
| `coordinates` | str (JSON) | Coordinates `{lat, lon}` | 0 |
| `has_flightable_airport` | bool | Has an airport | 0 |

### 5. Airport Directory — `api_airports.csv`

Full airport reference. **10,354 records, 9 features.**

| Feature | Type | Description | Missing |
|---------|------|-------------|---------|
| `name_translations` | str (JSON) | Names in multiple languages | 0 |
| `city_code` | str (IATA) | City code | 0 |
| `country_code` | str (ISO) | Country code | 34 |
| `time_zone` | str | Time zone | 0 |
| `code` | str (IATA) | IATA airport code | 0 |
| `iata_type` | str | Type (`airport`, `heliport`) | 0 |
| `name` | str | Name (English) | 0 |
| `coordinates` | str (JSON) | Coordinates `{lat, lon}` | 0 |
| `flightable` | bool | Available for flights | 0 |

### 6. Hotels (Booking.com) — `booking_hotels_all_cities.csv`

Hotel data collected by scraping Booking.com search results. **19,166 records, 7 features, 25 cities.**

| Feature | Type (raw) | Description | Format / Example | Missing |
|---------|------------|-------------|------------------|---------|
| `city` | str | City | `London`, `Rome`, `Paris` | 0 |
| `hotel_name` | str | Hotel name | `The 29 London` | 0 |
| `url` | str | Booking.com link | `https://www.booking.com/hotel/...` | 0 |
| `distance_to_center` | str | Distance to city center | `2.2 km from centre` | 0 |
| `price` | str | Price for 14 nights | `14,220 zl` (Polish zloty) | 0 |
| `rating_score` | str | Rating (0–10) | `Scored 7.5` | 413 (2.2%) |
| `review_count` | str | Number of reviews | `Good 3,099 reviews` | 416 (2.2%) |

**Hotel distribution across 25 cities:**

| City | Count | City | Count | City | Count |
|------|-------|------|-------|------|-------|
| Berlin | 848 | Copenhagen | 800 | Brussels | 750 |
| Tivat | 825 | Belgrade | 800 | Barcelona | 725 |
| Amsterdam | 816 | Warsaw | 800 | Madrid | 691 |
| Budapest | 812 | Vienna | 799 | Rome | 670 |
| Larnaca | 811 | Chisinau | 791 | Milan | 668 |
| Zurich | 802 | Stockholm | 789 | Athens | 536 |
| Helsinki | 800 | London | 780 | | |
| | | Prague | 779 | | |
| | | Paris | 775 | | |
| | | Bucharest | 774 | | |
| | | Dublin | 766 | | |
| | | Lisbon | 759 | | |

**Missing values in `rating_score` and `review_count`** (413–416 records, ~2.2%): new hotels without reviews or ratings on Booking.com. Represented as standard `NaN`.

**Raw data format notes (parsed to numeric during cleaning):**
- `price` — text with thousands separator and currency symbol (`14,220 zl`), parsed to float
- `rating_score` — contains `Scored` prefix (`Scored 7.5`), parsed to float (0–10)
- `review_count` — contains text rating and number (`Good 3,099 reviews`), parsed to int
- `distance_to_center` — contains unit of measurement (`2.2 km from centre` or `250 m from centre`), parsed to float (km)

---

## Data Cleaning (Part 3 in project.ipynb)

### Flights
- `price` converted to numeric type (`pd.to_numeric`)
- `departure_at`, `return_at` converted to datetime (UTC)
- Duplicates and rows without price are removed

### Hotels
- Text fields parsed to numeric using regular expressions:
  - `distance_to_center` → `distance_km` (float, km; `m` converted to km)
  - `price` → `hotel_price` (float; currency symbols and spaces removed)
  - `rating_score` → `hotel_rating` (float, 0–10)
  - `review_count` → `hotel_reviews` (int)
- Empty strings replaced with `NaN`
- Duplicates by `(hotel_name, city)` and rows without price are removed

### Missing Value Handling

| Dataset | Field | NaN Count | Reason | Decision | Rationale |
|---------|-------|-----------|--------|----------|-----------|
| Flights | `price` | 0 | — | Drop row (if NaN) | Price is required to calculate tour cost |
| Flights | `return_at` | possible | One-way flight | Keep NaN | Informative value: indicates one-way trip |
| Hotels | `hotel_price` | 0 | — | Drop row (if NaN) | Price is required for tour calculation |
| Hotels | `hotel_rating` | 413 (2.2%) | New hotel, no reviews | Keep NaN | Median imputation would distort data |
| Hotels | `hotel_reviews` | 416 (2.2%) | Hotel without reviews | Keep NaN | Parsed from text; if absent = NaN |
| Hotels | `distance_km` | 0 | — | — | — |
| Directories | `country_code` | 31–34 | Territories without ISO code | Keep NaN | Reference data, not used in analytics |

**Sentinel values:** no special fillers are used in the raw data. All missing values are standard `NaN`. Empty strings (`""`) are replaced with `NaN` via `df.replace(r'^\s*$', np.nan, regex=True)`.

### Cleaned Data

| File | Records | Features |
|------|---------|----------|
| `clean_flights.csv` | 214 | Same 17 fields; price — numeric, dates — datetime |
| `clean_hotels.csv` | ~19,000 | `city`, `hotel_name`, `url`, `distance_km`, `hotel_price`, `hotel_rating`, `hotel_reviews` |
| `clean_grouped_prices.csv` | 234 | Same 18 fields; price — numeric |

---

## Tour Recommendations (Part 4 in project.ipynb)

Flights and hotels are merged via **inner join on city** (city code → city name mapping from `api_popular_european_destinations.csv`). Each flight is combined with every hotel in the same city.

**Target variable:** `total_price = flight_price + hotel_price` (EUR)

**Generated rankings:**
- **TOP-10 cheapest tours** — `df_tours.nsmallest(10, "total_price")`
- **TOP-10 best value for money** — metric: `value_for_money = (hotel_rating / total_price) * 1000`
- **City comparison** — average metrics: tour cost, flight price, hotel price, rating, number of hotels
- **Cheapest flight** — minimum price from grouped data (API endpoint 5)

## Installation and Usage

```bash
pip install requests pandas selenium numpy
```

Run `project.ipynb` sequentially (all 4 parts).

**Scraping requirements:** Google Chrome + compatible ChromeDriver version.

