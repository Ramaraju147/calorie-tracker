# Curated micronutrient list limited to eight, chosen by data coverage

The MVP's curated micronutrient subset is: Vitamin D, Calcium, Iron, Potassium, Vitamin B12, Vitamin C, Magnesium, Zinc. We picked these over other candidates (e.g. Vitamin K, Biotin, Iodine, Choline) because they're both generally recognized as "good to know" by a non-expert and have reliable coverage in USDA FoodData Central and/or Open Food Facts — per ADR-0001's database-first Food Lookup, a nutrient with poor source data would show as empty/missing often enough to undermine trust in the Day Summary. Fiber, sodium, and added sugar were considered but excluded: USDA and FDA don't classify them as micronutrients (vitamins/minerals) either, and they sit outside this app's tracked categories (calories, macros, micronutrients, water, weight) for MVP.

## Consequences

Vitamin D is FDA-mandatory on packaged-food labels (so Open Food Facts often has it) but has poor USDA analytical coverage — expect it to fall back to Open Food Facts or the AI Estimate more often than the other seven nutrients in this list.

The set is fixed for every Account in MVP; no per-Account customization of which nutrients are tracked.
