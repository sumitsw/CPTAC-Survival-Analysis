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
Kaplan-Meier survival analysis **handles censored data** in a way that ensures the survival probabilities are calculated accurately, even when not all participants experience the event (e.g., death). Let me explain how censored data affects the calculation and why it leads to the survival probability dropping below 50%, even if fewer than 50% of participants have experienced the event:

---

### How Kaplan-Meier Handles Censored Data:
1. **Censored Data Doesn't Reduce the Denominator Immediately**:
   - Censoring occurs when a participant **leaves the study** or their follow-up time ends before they experience the event.
   - When a participant is censored, they are **still considered at risk** up to the point where they were last observed, but they do not contribute to events occurring after that point.
   - For survival probability calculations at a given time \( t \), censored participants are excluded **only after that time point**.

2. **Survival Probability Calculation**:
   - Kaplan-Meier calculates the survival probability step by step, at each event time:
     \[
     S(t) = S(t-1) \times \left( \frac{\text{Number at risk just before time } t - \text{Number of events at } t}{\text{Number at risk just before time } t} \right)
     \]
   - The presence of censored data reduces the number of participants **at risk** over time but doesn’t immediately affect the survival probability calculation.

---

### Example:
Let’s use an example with a small group of 10 participants:

| Time | Events (E) | Censored (C) | At Risk (N) | Survival Probability \( S(t) \) |
|------|------------|--------------|-------------|---------------------------------|
| 0    | 0          | 0            | 10          | 1.00                            |
| 5    | 1          | 0            | 10          | \( 1 \times \frac{9}{10} = 0.90 \) |
| 10   | 0          | 1            | 9           | \( 0.90 \) (no change, censoring only reduces \( N \)) |
| 15   | 1          | 0            | 8           | \( 0.90 \times \frac{7}{8} = 0.787 \) |

Here’s what happens:
- At **time 10**, one person is censored. This reduces the **at-risk population** for future events, but the survival probability doesn’t drop at this point because no event occurred.
- At **time 15**, an event happens, and the survival probability drops.

---

### Impact on Median Survival Time:
Now consider a scenario where many participants are censored before the survival probability drops to 50%. The survival curve will drop at a slower rate (because fewer events occur relative to the number of participants still at risk), but the censored data ensures that the curve still declines over time.

In your data:
- There are **28 total participants** but **only 8 events**.
- Despite fewer than 50% of participants experiencing the event, the survival probability \( S(t) \) eventually drops below 50% because the Kaplan-Meier method continuously updates \( S(t) \) as events occur and participants are censored.

---

### Why Median Survival Time is Affected:
Censoring changes the **denominator (number at risk)** in the survival probability calculation at each step:
- The survival probability can drop below 50% even if fewer than half the participants have experienced the event because censored participants reduce the denominator but are not treated as events.
- Thus, the **median survival time is based on the survival curve**, not a direct count of events.

---

### In Summary:
Kaplan-Meier accounts for censored participants by considering them at risk until their censoring time, which ensures the survival probabilities reflect the actual risk of experiencing the event. This handling of censored data allows the survival probability to drop below 50%, even if fewer than 50% of the participants experience the event directly.
