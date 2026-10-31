University Courses Enrichment - README
==============================================

WHAT THIS IS
A real, named list of courses/programs offered by ~1,655 US universities
in your universities.json, sourced from the U.S. Dept of Education
College Scorecard's field-of-study dataset (not estimated, not scraped -
official government data on actual conferred degrees).

FILES INCLUDED
- universities_enriched.json   -> your universities.json with courses_offered
                                   and courses_source added to every matched
                                   US university. THIS IS THE MAIN DELIVERABLE -
                                   swap this in for your current universities.json,
                                   or merge the new fields into your DB.
- enrich_courses.py            -> the script that produced it. Re-runnable if
                                   your universities.json changes.
- courses_by_unitid.json.gz    -> raw lookup (UNITID -> full course list,
                                   ungrouped, un-ranked). Gzipped, ~1.3MB.
                                   Only needed if you want to re-run/modify
                                   enrich_courses.py yourself.
- college_courses_trimmed.csv  -> the trimmed source CSV courses_by_unitid.json
                                   was built from (18MB). Only needed for
                                   re-processing from scratch.
- Most-Recent-Cohorts-Institution.csv -> same institution file from the
                                   earlier tuition/ranking enrichment, reused
                                   here for name matching.

WHAT EACH UNIVERSITY RECORD NOW HAS
  "courses_offered": [
    {
      "name": "Computer Science",
      "cip_code": "1107",
      "levels_offered": ["Bachelor's", "Master's", "Doctoral"],
      "total_grads_per_year": 722
    },
    ...
  ],
  "courses_source": "college_scorecard"   // or "unavailable" if unmatched

Courses are sorted by total_grads_per_year descending - the programs a
school actually produces the most graduates in show up first. Capped at
top 25 programs per university (change MAX_COURSES_PER_UNI in the script
to adjust).

MATCH RATE
1,655 of 10,259 universities matched (every US school in your dataset that
also exists in College Scorecard's institution list). International
universities are untouched (courses_source: "unavailable") - these should
fall through to your existing Firecrawl / AI-estimation fallback pipeline,
same data_source priority as the tuition/ranking enrichment:
curated -> college_scorecard -> firecrawl_scraped -> ai_estimated -> unavailable

HOW TO USE IN RYZE BACKEND
1. Swap universities_enriched.json in as your working dataset, or write a
   migration that merges courses_offered + courses_source into your existing
   MongoDB university documents by matching on university name.
2. For universities with courses_source: "unavailable", trigger your existing
   Firecrawl enrichment flow (scrape the university's admissions/programs
   page) the first time a user opens/searches that university, same pattern
   as tuition/ranking enrichment.
3. Use courses_offered in your Match Score engine's subject-relevance
   scoring - compare a student's stated field of interest against the
   university's actual top programs (by total_grads_per_year) instead of
   just the broad PCIP category percentages used before.

NOTE
This does NOT include international (non-US) universities - College
Scorecard is a US-only federal dataset. Non-US course data still needs
Firecrawl scraping or AI estimation per your existing enrichment pipeline.
