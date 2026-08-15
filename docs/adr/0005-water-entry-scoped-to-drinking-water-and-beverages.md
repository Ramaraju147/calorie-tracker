# Water Entry scoped to drinking water and beverages, not total water

The Water Nutrient Target could have been scoped three ways: plain water only, plain water plus beverages, or total water including food. We chose plain water plus beverages — a dedicated **Water Entry** for logging plain water (still, sparkling, or electrolyte-added), plus the water content of any beverage (milk, juice, coffee, tea, soda) logged as a Food Entry — and excluded solid-food water (soup, watermelon, yogurt) even though it's part of total water intake.

This maps exactly to NASEM's 2005 DRI category "fluids (drinking water and beverages)" — 3.0 L/day for adult men — which is distinct from, and smaller than, the full total-water AI of 3.7 L/day that also includes food-derived water (see [issue #5 research](https://github.com/Ramaraju147/calorie-tracker/issues/5), branch `research/rda-baseline-values`). Choosing "fluids" over "total water" meant the existing 3.0 L figure applied directly, with no relabeling or new sourcing needed — the alternative (total water) would have required tracking food-derived water too, which isn't visible to an Account and isn't something USDA/Open Food Facts data makes easy to reason about at the Food Entry level.

## Consequences

Food Entries for beverages must carry water-content data (USDA FoodData Central's native "Water" field, g/100g, ≈1:1 with mL) so they can feed the Water Nutrient Target alongside direct Water Entries. AI Estimate must also supply water content when used as a Food Lookup fallback, per ADR-0001's existing pattern for Food Entry nutrient data.

Flavored water is deliberately excluded from Water Entry and logged as a beverage Food Entry instead — the distinguishing line is "unflavored," not "contains only H₂O," so sparkling and electrolyte-added water still count as a Water Entry.
