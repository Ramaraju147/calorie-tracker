# Add Fiber, Added Sugar, and Sodium as a separate Watched Nutrient category

ADR-0003 excluded Fiber, Sodium, and Sugar from the curated Micronutrient list on the grounds that USDA/FDA don't classify them as micronutrients. On reflection that reasoning conflated vocabulary with relevance: these three (plus Added Sugar specifically, not Total Sugar — the one public-health guidance and FDA labels put a daily limit on) are exactly the trio a person already watches day-to-day from any Nutrition Facts label. We're adding them back as a second fixed, three-item Nutrient Target category — Watched Nutrient — kept separate from Micronutrient so the glossary stays technically accurate (they aren't vitamins or minerals) while still being tracked for every Account.

## Consequences

Sodium already has reliable coverage in both USDA FoodData Central and Open Food Facts (it's one of the "good coverage" nutrients identified during ADR-0003's research, and is FDA-label-mandatory). Fiber and Added Sugar are both standard label fields and expected to have similarly reliable coverage, though this wasn't independently re-verified — worth a quick check if Food Lookup gaps show up for these two in practice.
