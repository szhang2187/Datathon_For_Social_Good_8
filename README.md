# Datathon_For_Social_Good_8
Our project for the datathon for social good in 2026 November

**Social Good Question**: How do the availability and quality (review ratings) of essential services vary with median salary across geography and demographics?

**Datasets to Use:**
- Google Maps Places 
- Google Maps Reviews
- Indeed Jobs
- TIGER/Line tract shapefiles (via pygris) --> **this gives us the tract boundaries**
- Demographics 

Type --	Use for ratings?
grocery, pharmacy, bank:	Yes
clinic/hospital:	Yes, but only the ~13% with 10+ reviews
food bank:	Too thin (about 45 places qualify)
bus stop, school:	No, availability only


IMPORTANT FOR TRACK B & C
- Reviews file only overlaps with 112 of our places, all clinics (2,638 reviews, 646 are 1-2 star). No pharmacies, groceries, or banks. Complaint analysis can only cover clinics and will be thin by income group. For rating vs. income, use the Places file’s rating_clean (enough_reviews = True). Is there a bigger reviews file, or should we ask a mentor?
- The 646 low reviews come from 92 clinics in 60 ZIPs (median 1 clinic per ZIP). So complaints can only be compared as pooled lower-income vs. higher-income groups, not by ZIP or county. Treat it as exploratory.
- places_clean.csv is in my Drive (89,455 rows, 7 service types, ZIP column, rating_clean, is_chain, is_site, enough_reviews). Ratings only usable for grocery, pharmacy, bank, clinic (schools, bus stops, and food banks are availability-only). For clinic counts, use is_site == True (listings cluster at shared addresses). The ZIP income table is next.
- zip_table.csv is ready (261 Bay Area ZIPs). It has median income, income_group (Q1-Q5), county, race, car-free and limited-English shares, sites_, listings_, sites_per_10k_, has_none_ and median_rating_ columns per service type. Use sites_ for clinics. For Tableau, set zip as a string with the ZIP Code geographic role. 12 ZIPs hit the $250,001 income cap (they stay in Q5, since `pd.qcut` ranks them at the top) and 2 have no income, so those 2 are excluded from the income groups (259 ZIPs remain).


## Data decisions

**Cleaning (notebook 01_clean_places)**
- Started with 147,429 Places rows and saved 89,455 in `places_clean.csv`. Dropped: 12,167 closed places, 0 duplicate place_ids, 1,441 with no address, 361 with no ZIP in the address, and 44,005 outside our 7 service types (`drop_log.csv`).
- ZIP is the 5 digits after "CA" or "California" in the address (ZIP+4 handled). Hand check: 0 of 20 wrong.
- Rating 0 means no reviews (77,188 of 77,203 zero-rated places have 0 reviews), so it was treated as missing (`rating_clean`). Ratings are only analyzed where a place has 10+ reviews.
- Categories were mapped by exact label (`category_map.csv`). Clinics include general medical, dental and mental health. Specialists, optometrists, physical therapy, convenience stores and preschools are excluded. Hand check: about 0-2 of 20 wrong per type.
- About 23% of pharmacy-labeled listings were individual pharmacists or non-pharmacies, so we kept business listings only.
- Clinic listings cluster at shared addresses (63,111 listings, about 26,170 distinct sites). Use `is_site` for counts. Bus stops share generic addresses, so each counts as a site.
- Chain = same cleaned name 5+ times within a service type (bus stops excluded).
- Schools (99.8% with no rating), bus stops and food banks have too few reviews, so they are availability-only.

**Reviews**
- The Reviews file overlaps with only 112 of our places, all clinics (2,638 reviews, 646 at 1-2 stars from 92 clinics in 60 ZIPs). Complaint analysis is exploratory and pooled by income group. Reviewer names were never loaded.

**Census ZIP table (notebook 02_census_zip)**
- ACS 2020-2024 5-year, by ZIP (ZCTA), for the 9 Bay Area counties. A ZIP spanning counties goes to the county with more land. Dropped 39 ZIPs under 1,000 people, leaving 261.
- 12 ZIPs hit the $250,001 income cap and 2 have no income, so income is analyzed as quintiles (259 ZIPs).

**Coverage check**
- Google Maps vs. our data for 3 ZIPs: [paste your counts]. [One sentence: no sign the scrape misses more places in lower-income ZIPs.]

## How to run

Run the notebooks in order. Each one reads the previous one's output from `datathon/data/processed/` on Google Drive (in Colab) or `data/processed/` (locally).

1. `01_clean_places.ipynb`: Google Maps Places + Reviews → `places_clean.csv`, `reviews_low.csv`, `category_map.csv`, `drop_log.csv`
2. `02_census_zip.ipynb`: Census ACS (needs a `Census_Key` Colab secret) + `places_clean.csv` → `zip_table.csv`
3. `03_analysis.ipynb`: all three files → charts in `figures/`, tables and `key_findings.md` in `results/`

## Analysis (notebook 03_analysis)

Unit: 259 Bay Area ZIPs with an income estimate, split into five equal-count income quintiles (Q1 = lowest median household income, Q5 = highest).

| Question | Measure | How the quintiles are compared |
|---|---|---|
| How many sites per 10k residents? | distinct sites (`is_site`) ÷ population × 10,000 | median ZIP per quintile with 95% bootstrap interval, plus pooled rate; Spearman correlation with ZIP income across all ZIPs (Holm-adjusted for 7 services) |
| How often does a quintile have none? | `has_none_` (0 sites in the ZIP) | share of ZIPs and residents in ZIPs with none; Cochran-Armitage trend test Q1→Q5 (Holm-adjusted) |
| What median rating does each quintile get? | `rating_clean` for places with 10+ reviews (grocery, pharmacy, bank, clinic) | median, mean and low-rated share (at or below the Bay Area 25th percentile for that type); Q5−Q1 mean gap with a 95% interval that resamples ZIPs; robustness checks: no chains, 25+ reviews |
| What do clinic complaints say? (exploratory) | 1-2 star clinic reviews (`reviews_low.csv`) | lower (Q1-Q2) vs. higher (Q4-Q5) income ZIPs, Q3 dropped; share of reviews per keyword theme (Fisher's exact test, Holm-adjusted); distinctive words |

The full methodology and limitations are at the end of the notebook. Outputs:
- `figures/01_availability_by_quintile.png`, `02_has_none_by_quintile.png`, `03_ratings_by_quintile.png`, `04_rating_gap_q5_minus_q1.png`, `05_complaint_themes.png`
- `results/*.csv` (one table per chart, ready for Tableau) and `results/key_findings.md` (slide-ready sentences with numbers)

### Findings

Q1 = lowest-income fifth of ZIPs (median household income $86,694), Q5 = highest ($222,986), about 52 ZIPs each. "Clear" means Holm-adjusted p < 0.05 or a 95% interval that excludes 0. Full numbers: `results/key_findings.md`.

**Availability (sites per 10,000 residents, median ZIP)**
- **Groceries are more common in lower-income ZIPs:** 3.8 per 10k in Q1 vs. 1.5 in Q5 (Spearman rho −0.41, clear). But Q1 groceries are mostly small independents (13% chains vs. 42% in Q5) with far fewer reviews (median 48 vs. 358). The count measures stores, not full-service supermarkets.
- **Food banks are concentrated where need is higher:** 0.34 per 10k in Q1 vs. 0 in Q5 (rho −0.34, clear). 38% of Q1 ZIPs have none, vs. 63% of Q5 ZIPs.
- **Pharmacies, clinics, banks, bus stops and schools show no clear income trend** after adjustment. Banks go from 1.1 (Q1) to 2.3 (Q5) per 10k, and clinics from 20 to 33, but neither is statistically clear (p 0.20 and 0.07).

**ZIPs with none of a service**
- There is no income trend in which ZIPs lack a grocery, pharmacy, clinic or bank. For example, 23% of Q1 ZIPs vs. 29% of Q5 ZIPs have no pharmacy.
- 12 of 52 Q1 ZIPs lack at least one of the four (`results/02_q1_zips_missing_services.csv`).

**Ratings (places with 10+ reviews)**
- **Clinics in lower-income ZIPs rate lower:** mean 4.35 in Q1 vs. 4.65 in Q5 (gap +0.29, 95% CI +0.19 to +0.41). 37% of Q1 clinics are low-rated vs. 17% of Q5 clinics. The gap holds with chains removed (+0.30) and with a 25+ review cutoff (+0.25).
- **Pharmacies in higher-income ZIPs rate lower** (3.84 in Q1 vs. 3.41 in Q5), but the gap disappears for independent pharmacies (+0.06). It comes from chain mix: chain pharmacies, which rate low, are 47% of Q1 pharmacies vs. 70% of Q5.
- **Groceries and banks show no rating gap** (+0.01 and −0.06 stars).

**Context:** Q1 ZIPs are 39% Hispanic and 11% Black (vs. 10% and 2% in Q5), and 16% of Q1 households have no car (vs. 6%). These patterns are associations with a bundle of neighborhood traits, not with income alone.

**Not yet run:** the clinic-complaint comparison needs `reviews_low.csv`, which isn't in the repo yet. Notebook 03 skips that section until the file is added.

**Data caveat:** the scrape has only 782 pharmacy sites for 7.6 million residents, which looks low, so pharmacy coverage may be incomplete. The coverage check above is still pending.
