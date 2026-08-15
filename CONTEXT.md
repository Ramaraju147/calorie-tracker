# Calorie Tracker

A personal, invite-only, multi-tenant web app for tracking calories, macronutrients, a curated set of micronutrients, water, and body weight against daily targets.

## Language

**Account**:
An invited person's isolated space in the app — their own Food Log, Nutrient Targets, and Weight Entries, invisible to other Accounts.
_Avoid_: Tenant, User, Profile

**Food Log**:
The record of everything an Account has eaten on a given Day, grouped into Meals.
_Avoid_: Diary, Journal

**Meal**:
A named grouping (Breakfast, Lunch, Dinner, Snack) that Food Entries are logged under within a Day.
_Avoid_: Meal slot

**Food Entry**:
A single logged food item within a Meal, carrying its calories, macros, and micronutrients — sourced from a Food Lookup or an AI Estimate.
_Avoid_: Log entry, item

**Food Lookup**:
A search against USDA FoodData Central or Open Food Facts to find a Food Entry's nutrient values.
_Avoid_: Food search, database lookup

**AI Estimate**:
The nutrient values Claude generates for a Food Entry when a Food Lookup finds no match.
_Avoid_: AI guess, estimation

**Day**:
The single-day unit an Account's Food Log, Water Entries, and progress are tracked against; closes into a Day Summary.
_Avoid_: Date, session

**Day Summary**:
The end-of-day comparison of everything logged on a Day against that Day's Nutrient Targets, with an AI Recommendation explaining where the Account fell short or went over.
_Avoid_: Daily recap, close-out, end-of-day report

## Targets

**Nutrient Target**:
The RDA-based daily goal for a single nutrient (a macro, a micronutrient, calories, or water) that an Account's Day is measured against. Adjustable per Account.
_Avoid_: Goal, RDA, limit

**Micronutrient**:
One of a fixed, curated set of eight vitamins and minerals tracked as Nutrient Targets for every Account — Vitamin D, Calcium, Iron, Potassium, Vitamin B12, Vitamin C, Magnesium, Zinc. Chosen for general-population relevance and reliable Food Lookup data coverage (see [ADR-0003](docs/adr/0003-curated-micronutrient-list-limited-to-eight.md)).
_Avoid_: Vitamin, mineral, nutrient

**Calorie Target**:
The Nutrient Target for calories specifically — seeded from an Account's TDEE and Goal, and recalculated weekly from their Weight Trend.
_Avoid_: Calorie goal, budget

**AI Recommendation**:
The explanation and food suggestions Claude generates for a Day Summary, grounded in that Day's Nutrient Target comparison — never the source of the targets themselves.
_Avoid_: AI insight, AI advice

## Weight & goals

**Weight Entry**:
A single logged body-weight reading for an Account, timestamped, independent of any Day's Food Log.
_Avoid_: Weigh-in

**Weight Trend**:
The direction and rate of change across an Account's Weight Entries over time, used to recalculate their Calorie Target.
_Avoid_: Weight history

**Goal**:
An Account's stated intent — lose, maintain, or gain weight — at a chosen or suggested rate, used with TDEE to seed and adjust the Calorie Target.
_Avoid_: Objective, target (conflicts with Nutrient Target)

**TDEE**:
Total Daily Energy Expenditure — an Account's estimated maintenance calories, computed from their profile (age, sex, height, weight, activity level) via the Mifflin-St Jeor formula, and used to seed the initial Calorie Target.
_Avoid_: Maintenance calories, BMR
