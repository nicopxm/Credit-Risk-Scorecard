# Model Card — Credit Risk Scorecard
**Home Credit Default Risk · Phase 3 Final Model**

---

## Model details

| Field | Detail |
|---|---|
| Model type | Logistic Regression |
| Framework | scikit-learn 1.x |
| Encoding | Weight of Evidence (WoE) |
| Features | 14 WoE-encoded features |
| Output | Probability of Default (PD) between 0 and 1 |
| Decision threshold | 0.442 |
| Author | Nico Avila |
| Date | March 2026 |
| Version | 1.0 — Phase 3 Final |

---

## Intended use

**Primary use case:** Automated credit scoring at the point of loan application
for a digital consumer lending context.

**Intended users:** Credit managers, risk officers, and data analysts evaluating
applicant creditworthiness using a transparent, auditable scoring framework.

**Out of scope:** This model is not intended for:
- Business or commercial lending decisions
- Post-approval behavioural monitoring
- Any deployment without compliance and fairness review

---

## Training data

| Field | Detail |
|---|---|
| Dataset | Home Credit Default Risk (Kaggle) |
| Source | kaggle.com/competitions/home-credit-default-risk |
| Size | 307,511 loan applications |
| Features | 122 raw features → 14 selected via WoE/IV analysis |
| Target | Binary — 1 = default, 0 = repaid |
| Default rate | 8.1% (class imbalance addressed via class_weight='balanced') |
| Currency | Unconfirmed — all financial impact figures presented as ratios |
| Vintage | Unknown — application dates not provided in dataset |

---

## Model architecture — three phase development

The Final model was developed iteratively across three phases:

| Phase | Features | AUC | Gini | KS | Status |
|---|---|---|---|---|---|
| Phase 1 | 8 application features | 0.6498 | 0.2996 | 22.50 | Below target |
| Phase 2 | 11 (+ EXT_SOURCE bureau scores) | 0.7394 | 0.4787 | 36.27 | AUC gap |
| Phase 3 Final | 14 (+ behavioural and macro) | 0.7406 | 0.4812 | 35.97 | Conditional pass |

---

## Feature summary

| Feature | Type | IV | Business rationale |
|---|---|---|---|
| EXT_SOURCE_3_WOE | Bureau score | 0.3138 | Strongest external creditworthiness signal |
| EXT_SOURCE_2_WOE | Bureau score | 0.3063 | Composite credit quality indicator |
| EXT_SOURCE_1_WOE | Bureau score | 0.1360 | Supplementary bureau signal |
| EXT_SOURCES_MEAN_WOE | Engineered | — | Composite average — bridges thin-file gaps |
| EMPLOYMENT_YEARS_WOE | Stability | 0.0969 | Job tenure proxies income stability |
| AGE_YEARS_WOE | Demographic | 0.0821 | Younger applicants default at higher rates |
| EMPLOYED_TO_AGE_RATIO_WOE | Interaction | 0.0782 | Career stability relative to life stage |
| NAME_INCOME_TYPE_WOE | Categorical | 0.0597 | Income source stability signal |
| NAME_EDUCATION_TYPE_WOE | Categorical | 0.0508 | Education level — strongest categorical predictor |
| PAYMENT_RATE_WOE | Ratio | 0.0449 | Repayment speed relative to principal |
| DAYS_LAST_PHONE_CHANGE_YEARS_WOE | Behavioural | 0.0467 | Lifestyle stability proxy |
| CODE_GENDER_WOE | Demographic | 0.0388 | Gender differential — fairness flag applied |
| REGION_RATING_CLIENT_WOE | Macro | 0.0276 | Geographic credit risk environment |
| NAME_FAMILY_STATUS_WOE | Categorical | 0.0218 | Marital stability proxy |

---

## Performance metrics

### Technical metrics (holdout test set — 61,503 applicants)

| Metric | Value | Target | Status |
|---|---|---|---|
| AUC | 0.7406 | > 0.75 | ⚠ Conditional pass |
| Gini coefficient | 0.4812 | > 0.40 | ✓ Pass |
| KS statistic | 35.97 | > 30 | ✓ Pass |

### Business metrics (at optimal threshold 0.442)

| Metric | Value | Target | Status |
|---|---|---|---|
| Approval rate | 55.0% | ≥ 55% | ✓ Pass |
| Portfolio default rate | 3.37% | ≤ 7.5% | ✓ Pass |
| Safety buffer | 4.13% below ceiling | — | Strong |

### Risk tier performance

| Tier | Applicants | Default rate | Decision |
|---|---|---|---|
| 1 — Very Low (Prime) | 8,493 | 1.67% | Approve — best pricing |
| 2 — Low (Standard) | 8,493 | 2.78% | Approve — standard terms |
| 3 — Medium (Conditional) | 16,986 | 4.52% | Approve — manual review |
| 4 — High (Decline) | 27,531 | 13.88% | Decline |

---

## Fairness assessment

### Gender differential

| Gender | Default rate | Relative risk |
|---|---|---|
| Female | 7.0% | Baseline |
| Male | 10.1% | 1.44x higher |
| XNA (4 records) | 0.0% | Excluded from analysis |

**Finding:** Male applicants default at a 44% higher relative rate than female
applicants in this dataset. CODE_GENDER is included as a model feature based
on its IV of 0.0388 — above the 0.02 selection threshold.

**Action required before production deployment:**
- Legal and compliance review of gender as a direct model input
- Assessment against applicable fair lending regulations
(ECOA in the US, Consumer Credit Directive in the EU,
CONDUSEF guidelines in Mexico)
- Consideration of adverse impact analysis across protected classes
- Sign-off from Compliance / Legal stakeholder (per BRD Section 8)

---

## Limitations and risks

| Limitation | Detail | Mitigation |
|---|---|---|
| AUC marginally below target | 0.7406 vs 0.75 target | Conditional pass — business KPIs exceeded |
| Class imbalance | 8.1% default rate | class_weight='balanced' applied |
| Currency unconfirmed | Dataset denomination unknown | All impact figures presented as ratios |
| Dataset vintage unknown | Application dates not provided | Recommend revalidation under new conditions |
| EXT_SOURCE methodology undisclosed | Bureau score composition proprietary | Treat as composite signal — verify with provider |
| Single time snapshot | No economic cycle variation | Model may degrade under different macro conditions |
| Gender fairness concern | 44% higher male default rate | Compliance review required before deployment |
| Logistic regression ceiling | Non-linear patterns may be missed | XGBoost benchmark recommended as future iteration |

---

## Regulatory considerations

This scorecard is developed using logistic regression on WoE-encoded features
to satisfy the following regulatory and governance requirements:

- **Explainability** — every feature has a documented business rationale
  and a directly interpretable coefficient (log-odds contribution)
- **Adverse action notices** — the model's linear structure supports
  generating reason codes for declined applicants as required by
  fair lending regulations
- **Auditability** — full reproducibility from the published GitHub
  repository (FR-07)
- **Fairness documentation** — gender differential flagged and documented
  pending compliance review

---

## Recommendations for production deployment

1. **Soft launch** — deploy with a 90-day Final-Challenger monitoring
   period tracking Population Stability Index (PSI) and default rate stability
2. **Threshold review** — reassess the 0.442 threshold quarterly against
   actual portfolio default rates
3. **Gender compliance** — complete legal review of gender feature before
   live deployment
4. **Model revalidation** — retrain annually or when PSI exceeds 0.2
5. **Future iteration** — benchmark against XGBoost to quantify the
   explainability-accuracy tradeoff

---

## Future improvements

- Benchmark logistic regression against XGBoost and Random Forest
- Incorporate behavioural data from bureau and instalment payment tables
- Implement SHAP values for feature-level explainability on ensemble models
- Expand fairness assessment to additional protected characteristics
- Add Population Stability Index monitoring framework

---

## References

- Home Credit Default Risk dataset — kaggle.com/competitions/home-credit-default-risk
- Basel III consumer credit guidelines
- FCA affordability and LTI standards
- ECOA fair lending requirements
- scikit-learn LogisticRegression documentation

---

*This model card was prepared as part of a portfolio project demonstrating
credit risk scorecard development methodology. It is not intended for
actual production deployment without full regulatory review.*