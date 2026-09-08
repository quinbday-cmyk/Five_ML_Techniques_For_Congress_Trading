# Congressional Stock Trade Profitability Prediction

## Overview

This project asks a simple question: can publicly available, pre-trade information from congressional stock trade disclosures predict whether a given trade will turn out to be profitable? Each disclosure comes with rich contextual metadata — the trading legislator's party, chamber, committee memberships, and home state — making these records a natural testbed for whether such features correlate with, and can help forecast, ex-post profitability.

## Background: The STOCK Act

The Stop Trading on Congressional Knowledge (STOCK) Act of 2012 requires members of Congress to disclose individual securities transactions — the instrument traded, an approximate dollar range, and trade direction (buy/sell) — within 45 calendar days. In practice, enforcement has been essentially nonexistent: the maximum penalty is a $200 fine, and the Act has never been enforced.

A 2013 amendment — passed by unanimous consent in both chambers (with just 14 seconds of House floor discussion) and signed into law four days after introduction — allows members to disclose privately rather than publicly. Whether this materially changes disclosure behavior is unclear, since the underlying Act hasn't been enforced either way.

## Research Question

Does congressional trading show a detectable statistical pattern beyond passive market exposure? This matters for a few reasons:

- If certain legislators trade profitably in sectors they oversee, it may point to an information advantage — from committee work, industry ties, or regulatory awareness — being converted into personal investment returns.
- Persistently profitable trading in regulated sectors (defense, healthcare, financial services) could represent a form of informed trading with downstream implications for price discovery.
- Even a modest predictive edge is informative if it's concentrated in specific legislators, committees, or sectors, since that concentration reveals where the disclosure data actually carries signal.

## Prediction Task

The task is binary classification. The target, `Is_Profitable_Signal`, equals 1 when a trade's direction is retrospectively validated:

- A **buy** is labeled profitable if the stock price rose over the following **180 days**.
- A **sell** is labeled profitable if the stock price fell over the following **90 days**.

The asymmetric windows reflect an assumption about trading intent: buys are treated as longer-horizon holds (180 days also accommodates the relatively short 2023–2026 span of the CapitolTrades dataset, plus the lag before committee-related information becomes public and gets priced in), while sells are treated as shorter-horizon exits from positions expected to underperform soon.

**Methodological constraint:** every feature must be observable at the time of the trade. No post-trade prices, returns, or outcomes are permitted anywhere in the feature matrix. Enforcing this temporal discipline was the central methodological focus of the project — it's the easiest place for lookahead bias to creep into an otherwise reasonable-looking model.

## Results

**Three-class models**
- MLP: best performer, 61% test accuracy, macro F1 of 0.58.
- Logistic regression: did not meaningfully beat baseline.
- The gap suggests the MLP's edge came from capturing non-linear relationships, and likely also from being comparatively robust to overfitting on a small dataset.

**Two-class models**
- Random forest: best performer — F1 of 0.554, AUC of 0.596 — again outperforming simpler linear models (e.g., linear SVM) by capturing more complex relationships.

## Discussion

The results point to a real but weak signal in congressional stock disclosures — present, but not strong enough on its own to support a reliable trading strategy. A few factors likely limit the signal:

- **Disclosure lag** — trades aren't reported for up to 45 days, by which point any information edge is plausibly already priced in.
- **Coarse trade sizing** — transaction amounts are reported as ranges, not exact dollar figures.
- **Uneven activity** — trading volume varies widely across legislators and over time, making the dataset harder to model consistently.
- **Non-contemporaneous data** — the disclosure process introduces a structural gap between when a trade happens and when it becomes observable.

## Future Work

- Apply NLP to legislators' public statements to capture new information closer to real time.
- Add macroeconomic and other financial data to the MLP and random forest models (the strongest performers here) to see if it moves results toward a genuinely profitable strategy.
- Run all models across both the two-class and three-class formulations for a more complete comparison.
