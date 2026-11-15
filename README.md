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

- Reviews file only overlaps with 112 of our places, all clinics (2,638 reviews, 646 are 1-2 star). No pharmacies, groceries, or banks. Complaint analysis can only cover clinics and will be thin by income group. For rating vs. income, use the Places file’s rating_clean (enough_reviews = True). Is there a bigger reviews file, or should we ask a mentor?
- The 646 low reviews come from 92 clinics in 60 ZIPs (median 1 clinic per ZIP). So complaints can only be compared as pooled lower-income vs. higher-income groups, not by ZIP or county. Treat it as exploratory.
- places_clean.csv is in my Drive (89,455 rows, 7 service types, ZIP column, rating_clean, is_chain, is_site, enough_reviews). Ratings only usable for grocery, pharmacy, bank, clinic (schools, bus stops, and food banks are availability-only). For clinic counts, use is_site == True (listings cluster at shared addresses). The ZIP income table is next.
