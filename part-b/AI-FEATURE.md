# AI Feature Specification: Smart Vikalp Predictor
 
## Problem It Solves
This feature addresses **Problem 5 (Vikalp Scheme UI is Confusing)** and mitigates **Problem 1 (Tatkal Crashes)**. By dynamically predicting and suggesting high-probability alternate trains, it transforms the confusing Vikalp checkbox into a proactive, intelligent travel planning assistant that steers traffic away from overloaded trains.
 
## Proposed Feature — User Perspective
When a user searches for a train and views a heavily Waitlisted (WL) option, an inline AI card appears below the train details. It says: **"Smart Route: 85% chance of confirmation on Train 12345 departing 3 hours later."** The user can click a single button to switch their booking context to the suggested train, significantly improving their chances of a confirmed ticket without having to guess which Vikalp options are viable.
 
## Model or API Choice
**Google Vertex AI Tabular Data Model (AutoML)**. 
*Why:* IRCTC possesses massive amounts of structured, historical tabular data (PNR statuses, train capacities, seasonal booking velocities). Vertex AI excels at time-series forecasting and tabular classification without requiring a massive custom ML engineering team. It can easily predict the binary outcome (Confirm vs. Remain Waitlisted) based on current parameters.
 
## Training or Input Data
**Data required by the model:**
- Historical waitlist movement data for specific routes and times.
- Current waitlist velocity (how fast tickets are being booked/cancelled right now).
- Seasonality flags (festivals, weekends, holidays).
- Train capacity and class quotas.

**Where it comes from:**
- IRCTC's internal booking database (historical PNR datasets).
- Real-time inventory APIs.
*This data is fully available internally to IRCTC and simply needs to be pipelined to the Vertex AI endpoints.*
 
## How Output Is Shown to the User
The output is presented as an inline suggestion chip directly beneath a Waitlisted class option in the search results.

```text
=========================================
Train: 12951 | RAJDHANI EXP
Class: 3A | Status: WL 145 / WL 90
-----------------------------------------
✨ AI Suggestion: Low confirmation chance.
👉 Switch to 12953 (Departs 4h later) for an 85% confirmation chance.
[ Switch Train ]
=========================================
```
 
## Confidence Threshold and Fallback
- **Threshold:** The AI output is only shown if it finds an alternate train on the same route with a confirmation probability of **> 75%**, AND the original train's confirmation probability is **< 30%**.
- **Fallback:** If the confidence threshold is not met, or the Vertex AI API times out, the AI suggestion card remains completely hidden. The user simply sees the standard waitlist numbers and proceeds with the normal flow.
 
## Success Metrics
- **Conversion to Alternates:** Percentage of users who click "Switch Train" from the AI suggestion.
- **Post-Booking Confirmation Rate:** Percentage of users who used the AI feature and successfully received a confirmed chart status.
- **Load Balancing:** Reduction in waitlist congestion on flagship trains.
 
## Limitations and Risks
- **Overpromising:** If the model predicts an 85% chance but the ticket remains waitlisted, user trust will plummet.
- **Data Drift:** Festival rushes or sudden train cancellations can invalidate historical patterns, causing the model to hallucinate false probabilities.
- **Risk Mitigation:** Always display probabilities as "Estimated" and never guarantee confirmation. Regularly retrain the model on rolling 30-day data to capture sudden trend shifts.
