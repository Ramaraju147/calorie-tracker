# TDEE activity multipliers and safe rate-of-change guidance

Research for [issue #6](https://github.com/Ramaraju147/calorie-tracker/issues/6), part of the wayfinder map [#1](https://github.com/Ramaraju147/calorie-tracker/issues/1).

Scope: (1) standard activity-multiplier bands used to turn BMR into TDEE under the Mifflin-St Jeor formula, and (2) credible guidance for a safe default rate of weight change for a "lose" or "gain" **Goal**, expressed so it can seed the **Calorie Target** and be recalculated weekly from **Weight Trend** (see `CONTEXT.md`).

---

## Recommended defaults (ready to hard-code)

| Goal | Default rate | Calorie adjustment from TDEE | Basis |
|---|---|---|---|
| Lose weight | **0.5 kg/week** (≈1 lb/week), bounded to a 0.45–0.9 kg/week (1–2 lb/week) safe range | **−500 kcal/day** (range −500 to −1000 kcal/day for the 1–2 lb/week band) | CDC + Academy of Nutrition and Dietetics (AND) |
| Gain weight | **0.25% of bodyweight/week**, bounded to a 0.25–0.5%/week range | **+250 kcal/day** (range +250 to +500 kcal/day; do not exceed +500) | Iraki et al. 2019 (peer-reviewed narrative review, *Sports*) |
| Maintain | 0%/week | 0 kcal/day (TDEE) | — |

Activity multiplier to seed TDEE from Mifflin-St Jeor BMR — use the conventional 5-band scale (sourced below), since it is what the Mifflin-St Jeor formula is paired with in essentially all clinical and consumer practice:

| Band | Multiplier | Real-world activity |
|---|---|---|
| Sedentary | **1.2** | Little to no exercise, desk job |
| Lightly active | **1.375** | Light exercise/sports 1–3 days/week |
| Moderately active | **1.55** | Moderate exercise/sports 3–5 days/week |
| Very active | **1.725** | Hard exercise/sports 6–7 days/week |
| Extra active | **1.9** | Very hard exercise, physical job, or training 2×/day |

Both tables and the reasoning behind the recommended defaults are detailed below, with primary-source citations.

---

## 1. Activity-level multiplier bands

### 1.1 The Mifflin-St Jeor equation itself only produces BMR, not TDEE

The original publication — Mifflin MD, St Jeor ST, Hill LA, Scott BJ, Daugherty SA, Koh YO. "A new predictive equation for resting energy expenditure in healthy individuals." *American Journal of Clinical Nutrition*. 1990;51(2):241–247. doi:[10.1093/ajcn/51.2.241](https://doi.org/10.1093/ajcn/51.2.241) — derived a regression equation for **resting energy expenditure (REE/BMR)** from a sample of healthy adults. It does **not** define activity multipliers; those were layered on afterward by later clinical/dietetic practice to convert BMR to TDEE. The equations:

- Men: BMR (kcal/day) = 10×weight(kg) + 6.25×height(cm) − 5×age(years) + 5
- Women: BMR (kcal/day) = 10×weight(kg) + 6.25×height(cm) − 5×age(years) − 161

A 2005 systematic review — Frankenfield DC, Roth-Yousey L, Compher C. "Comparison of predictive equations for resting metabolic rate in healthy nonobese and obese adults: a systematic review." *Journal of the American Dietetic Association*. 2005;105(5):775–789 — validated Mifflin-St Jeor as the most accurate of the widely used predictive equations for non-obese and obese adults, which is why it remains the standard BMR formula to pair with an activity multiplier for TDEE.

### 1.2 The conventional 5-band multiplier scale (1.2 / 1.375 / 1.55 / 1.725 / 1.9)

This is the scale used by virtually every clinical BMR/TDEE calculator and consumer nutrition app that pairs with Mifflin-St Jeor (e.g., the calculators referenced by Medscape/QxMD's Mifflin-St Jeor implementation). **Caveat on provenance**: unlike the BMR formula itself, this specific 5-tier scale does not trace to one peer-reviewed publication — it is a practitioner convention that has become the de facto standard for this exact use case (BMR × activity factor → TDEE). It should be understood as "widely adopted clinical/commercial practice," not a single-paper citation.

- Sedentary — ×1.2 — little to no exercise, desk job
- Lightly active — ×1.375 — light exercise/sports 1–3 days/week
- Moderately active — ×1.55 — moderate exercise/sports 3–5 days/week
- Very active — ×1.725 — hard exercise/sports 6–7 days/week
- Extra active — ×1.9 — very hard exercise/physical job, or training twice a day

### 1.3 Cross-check: the Institute of Medicine's Dietary Reference Intake (DRI) Physical Activity Level (PAL) bands

For a more rigorous, government-panel-derived cross-check, the National Academies' Institute of Medicine (IOM) *Dietary Reference Intakes for Energy, Carbohydrate, Fiber, Fat, Fatty Acids, Cholesterol, Protein, and Amino Acids* (2002/2005) defines Physical Activity Level (PAL) categories and uses them in its own Estimated Energy Requirement (EER) equations (a different equation form from Mifflin-St Jeor, built directly from weight/height/age/PA rather than as a BMR multiplier). Source: [Dietary Reference Intakes for Energy — Ch. 12, Physical Activity, National Academies Press](https://nap.nationalacademies.org/read/10490/chapter/14); summarized at [NCBI Bookshelf](https://www.ncbi.nlm.nih.gov/books/NBK591034/) and [CDC Preventing Chronic Disease, Oct 2006](https://www.cdc.gov/pcd/issues/2006/oct/06_0034.htm).

| PAL category | PAL range | EER activity coefficient (male / female, approx.) |
|---|---|---|
| Sedentary | 1.0 – <1.4 | 1.00 |
| Low active | 1.4 – <1.6 | ~1.11 / 1.12 |
| Active | 1.6 – <1.9 | ~1.25 / 1.27 |
| Very active | 1.9 – 2.5 | ~1.45 / 1.48 |

These PAL-band coefficients are **not directly interchangeable numbers** with the §1.2 BMR-multiplier scale (different base equation, and the IOM coefficient is applied inside a formula that already includes its own weight/height/age terms — it isn't a flat multiplier on a separately computed BMR). But qualitatively the two scales agree well: both top out with a "very active/extra active" ceiling around 1.9, both start at 1.0–1.2 for sedentary, and both space "light" and "moderate" activity similarly. This cross-check is why §1.2's 5-band scale is a defensible, standards-consistent choice for TDEE = BMR × multiplier, even though its exact numbers aren't drawn from the same source as the BMR formula.

**Recommendation**: use the §1.2 five-band scale (1.2/1.375/1.55/1.725/1.9) as the app's activity multiplier, since it's built for exactly this purpose (multiplying a Mifflin-St Jeor BMR) and is corroborated in shape by the IOM's independently-derived PAL bands.

---

## 2. Safe default rate of weight change

### 2.1 Losing weight

**CDC** — "Steps for Losing Weight," Healthy Weight and Growth: https://www.cdc.gov/healthy-weight-growth/losing-weight/index.html — states that people who lose weight at a gradual, steady pace of about **1 to 2 pounds a week** are more likely to keep the weight off long-term than people who lose weight quickly.

**Academy of Nutrition and Dietetics (AND)** — Adult Weight Management Evidence-Based Practice Guideline (andeal.org, Adult Weight Management archive: https://www.andeal.org/vault/pqnew118.pdf, https://www.andeal.org/vault/pqnew130.pdf) — recommends an optimal rate of **1–2 lb/week for the first 6 months**, achieved via a **caloric deficit of 500–1000 kcal/day** below estimated energy needs, targeting up to a 10% reduction from baseline weight in that period.

**The flat "3500 kcal = 1 lb" arithmetic behind these deficit numbers is a known oversimplification.** It originates from Wishnofsky M. "Caloric equivalents of gained or lost weight." *American Journal of Clinical Nutrition*. 1958;6(5):542–546 — a bomb-calorimetry estimate that adipose tissue stores ~3,500 kcal/lb. A static "500 kcal/day deficit forever = 1 lb/week forever" projection is inaccurate over time because it ignores metabolic adaptation (TDEE falls as body weight falls). NIH/NIDDK addressed this with a validated dynamic model, publicly available as the **NIH Body Weight Planner** (built on Hall KD et al.'s dynamic energy-balance model; NIDDK licensing page: https://www.niddk.nih.gov/research-funding/technology-advancement-transfer/research-materials-licensing/body-weight-simulator-java-applet). A 2012 American Society for Nutrition consensus recommended practitioners adopt this dynamic model and retire the flat 3500-kcal rule of thumb for long-run projections.

This matters directly for this app's design: per `CONTEXT.md`, the Calorie Target is **seeded** from TDEE/Goal and then **recalculated weekly from actual Weight Trend**. That weekly recalculation is exactly the corrective mechanism NIH's critique calls for — the static 500 kcal/day deficit is only ever used as the *initial* seed (Day 1), and the app self-corrects against the metabolic-adaptation error every week using real Weight Entries rather than trusting the flat arithmetic indefinitely. So using −500 kcal/day (≈1 lb/week) as the seed is safe and standards-aligned specifically because this app doesn't rely on it long-term.

**Recommended default**: −500 kcal/day (≈0.45 kg / 1 lb per week), within an overridable safe band of −500 to −1000 kcal/day (1–2 lb per week), consistent with both CDC and AND. Expressed as %bodyweight/week this is roughly 0.5–1%/week for a typical adult, scaling down for lighter individuals — but note CDC/AND state the guidance in absolute lb/week terms, not %bodyweight, so the %BW figure here is a derived convenience, not an independently-sourced number.

### 2.2 Gaining weight

There is no CDC/NIH public-health guideline for a safe rate of *weight gain* — gaining weight isn't a population health target the way weight loss is, so the strongest available peer-reviewed guidance comes from sports-nutrition literature rather than public-health agencies:

**Iraki J, Fitschen P, Espinar S, Helms E.** "Nutrition Recommendations for Bodybuilders in the Off-Season: A Narrative Review." *Sports*. 2019;7(7):154. doi:[10.3390/sports7070154](https://doi.org/10.3390/sports7070154) (open access, peer-reviewed, PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC6680710/). This narrative review (co-authored by sports-nutrition researcher Eric Helms) recommends:

- A hyper-caloric diet of roughly **+10–20% above maintenance**.
- A target weight gain rate of **~0.25–0.5% of bodyweight per week** for novice/intermediate trainees.
- **More conservative** (lower) rates for advanced trainees, since they have less capacity to build new lean tissue and are more prone to excess fat gain at the same surplus.
- Evidence that surpluses **larger than ~500 kcal/day** increase fat accumulation disproportionately more than they accelerate muscle growth — i.e., bigger surpluses don't buy proportionally more useful gain, just more fat.

**Recommended default**: **+250 kcal/day**, corresponding to roughly 0.25% of bodyweight/week for a typical adult — the conservative end of the Iraki et al. range, chosen as a universal default because it's safe for both novice and advanced trainees (advanced users should not exceed this without deliberately overriding it). Overridable range: +250 to +500 kcal/day (0.25–0.5%/week); the app should not suggest surpluses above +500 kcal/day given the fat-gain-disproportionality evidence above.

### 2.3 Converting rate → daily calories (for implementation reference)

- Fat loss: ~3,500 kcal ≈ 1 lb ≈ 0.4536 kg of fat tissue (Wishnofsky's calorimetry estimate, §2.1). A 500 kcal/day deficit → 3,500 kcal/week → ≈1 lb (≈0.45 kg)/week. This is a reasonable *seed* estimate; treat it as a starting point that the weekly Weight-Trend-based recalculation should correct, not as a fixed lifetime conversion factor (per the NIH critique in §2.1).
- Weight gain: no equally precise calorimetry constant exists, because gained tissue is a variable mix of lean mass and fat (unlike fat loss, which is compositionally simpler to model). Treat the +250 to +500 kcal/day figures from Iraki et al. as the primary source of truth for the gain side, rather than back-deriving them from a kcal/kg constant.

---

## Sources

1. Mifflin MD, St Jeor ST, Hill LA, Scott BJ, Daugherty SA, Koh YO. "A new predictive equation for resting energy expenditure in healthy individuals." *Am J Clin Nutr*. 1990;51(2):241–247. doi:10.1093/ajcn/51.2.241
2. Frankenfield DC, Roth-Yousey L, Compher C. "Comparison of predictive equations for resting metabolic rate in healthy nonobese and obese adults: a systematic review." *J Am Diet Assoc*. 2005;105(5):775–789.
3. Institute of Medicine. *Dietary Reference Intakes for Energy, Carbohydrate, Fiber, Fat, Fatty Acids, Cholesterol, Protein, and Amino Acids*, Ch. 12 "Physical Activity." National Academies Press, 2002/2005. https://nap.nationalacademies.org/read/10490/chapter/14 ; summary at https://www.ncbi.nlm.nih.gov/books/NBK591034/ ; CDC summary: https://www.cdc.gov/pcd/issues/2006/oct/06_0034.htm
4. CDC. "Steps for Losing Weight." Healthy Weight and Growth. https://www.cdc.gov/healthy-weight-growth/losing-weight/index.html
5. Academy of Nutrition and Dietetics. Adult Weight Management Evidence-Based Nutrition Practice Guideline. https://www.andeal.org/vault/pqnew118.pdf and https://www.andeal.org/vault/pqnew130.pdf
6. Wishnofsky M. "Caloric equivalents of gained or lost weight." *Am J Clin Nutr*. 1958;6(5):542–546. (Origin of the 3,500 kcal/lb estimate.)
7. NIDDK/NIH. Body Weight Simulator / Body Weight Planner (dynamic energy-balance model, Hall KD et al.), licensing/background page: https://www.niddk.nih.gov/research-funding/technology-advancement-transfer/research-materials-licensing/body-weight-simulator-java-applet
8. Iraki J, Fitschen P, Espinar S, Helms E. "Nutrition Recommendations for Bodybuilders in the Off-Season: A Narrative Review." *Sports*. 2019;7(7):154. doi:10.3390/sports7070154. PMC: https://pmc.ncbi.nlm.nih.gov/articles/PMC6680710/
