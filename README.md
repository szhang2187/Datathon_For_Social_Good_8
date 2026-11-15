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

Reviews file only overlaps with 112 of our places, all clinics (2,638 reviews, 646 are 1-2 star). No pharmacies, groceries, or banks. Complaint analysis can only cover clinics and will be thin by income group. For rating vs. income, use the Places file’s rating_clean (enough_reviews = True). Is there a bigger reviews file, or should we ask a mentor?
