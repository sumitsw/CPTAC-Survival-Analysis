### Kaplan-Meier Estimation and Median Survival
In Kaplan-Meier survival analysis:
1. The **survival probability \( S(t) \)** is calculated at each time point based on the number of individuals at risk and the number of events (deaths) that occur.
2. The **median survival time** is defined as the time when the survival probability \( S(t) \) drops to **50% or below**. It is **not based on counting deaths directly** but on the survival probability, which accounts for both deaths and censored data.

---
- The **median survival time** is the **time closest to when \( S(t) \) drops to or below 0.5**.
---

### Why 50% Deaths and Median Survival Time Don't Always Align
The **median survival time** in Kaplan-Meier analysis is based on survival probabilities, not a direct count of individuals who experienced the event. 

For example:
- If you start with 28 participants and only 8 experience the event, the survival probability \( S(t) \) can still drop below 50% due to the way Kaplan-Meier handles censored data.
- Censored data does not "reduce the denominator" in the same way as deaths (events), which means the survival probability can drop below 50% even if fewer than half of the participants experienced the event.

---
