# The Complete Forecaster's Journey — Exam Study Guide

> **Practical Time Series Forecasting** | Chapters 1–10 (Python Edition)  
> Prepared: February 20, 2026 | Read like a story, not a reference manual.

---

## The Big Picture — How Everything Connects

Imagine you've just been hired as a **forecast analyst** at Amtrak (the US railway company). Your boss walks in and says: *"We need to know how many passengers will ride our trains for the next 36 months."*

This entire course is the journey of answering that question — from collecting data, through building models, to presenting your forecasts to management. Here's how the chapters connect into one coherent story:

```
┌─────────────────────────────────────────────────────────────────────┐
│  THE FORECASTER'S WORKFLOW                                          │
│                                                                     │
│  1. DEFINE THE PROBLEM          (Ch 1: Approaching Forecasting)     │
│     ↓ "What are we forecasting? How far ahead? For whom?"           │
│  2. COLLECT & EXPLORE DATA      (Ch 2: Time Series Data)            │
│     ↓ "What does our data look like? What patterns exist?"          │
│  3. SET UP EVALUATION           (Ch 3: Performance Evaluation)      │
│     ↓ "How will we know if our model is actually good?"             │
│  4. BUILD MODELS (pick your tool):                                  │
│     ├─ Simple/Adaptive      → (Ch 5: Smoothing Methods)             │
│     ├─ Structured/Global    → (Ch 6: Regression — Trend & Season)   │
│     ├─ Autocorrelation-aware→ (Ch 7: AR, ARIMA, External Info)      │
│     ├─ Complex/Nonlinear    → (Ch 8: Neural Networks)               │
│     └─ Binary outcomes      → (Ch 9: Logistic Regression)           │
│  5. DEPLOY & COMMUNICATE        (Ch 10: Communication & Maintenance)│
│     "Present results, monitor, and keep forecasts updated."         │
└─────────────────────────────────────────────────────────────────────┘
```

Think of it this way: Chapters 1–3 are the **foundation** (what, why, and how to judge). Chapters 5–9 are your **toolbox** (increasingly sophisticated tools). Chapter 10 is the **delivery** (presenting your work to the real world). Each tool in the toolbox builds on the limitations of the previous one.

---

# Part I: The Foundation

---

## Chapter 1 — What Is Forecasting and Why Does It Matter?

### Starting From Scratch: What Is a Time Series?

Before we forecast anything, we need to understand what we're working with. A **time series** is simply a sequence of data points collected over time, at regular intervals.

**Everyday examples:**
- Your monthly electricity bill over the past 3 years
- Daily temperature in your city
- Hourly website traffic
- Quarterly revenue of a company
- Monthly ridership on Amtrak trains (our running example throughout)

The key difference from other data: **order matters**. January's value comes before February's. You can't shuffle time series data like you would a spreadsheet of customer records.

### Two Types of Data — A Critical Distinction

| | Cross-Sectional Data | Time Series Data |
|---|---|---|
| **What it is** | Snapshot at one point in time | Measurements over many time points |
| **Example** | Survey of 1000 customers today | Monthly sales for past 5 years |
| **Order matters?** | No (rows can be shuffled) | YES (temporal sequence is everything) |
| **Goal** | Predict for new individuals | Predict future values |

This distinction is fundamental and affects everything: how we split data, which models we use, and what "error" means.

### The Language of Forecasting — Notation

Let's establish the vocabulary we'll use throughout:

| Symbol | Meaning | Example |
|--------|---------|---------|
| `t = 1, 2, 3…` | Time period index | t=1 is Jan 1991, t=2 is Feb 1991, etc. |
| `Yₜ` | Actual value at time t | Y₁ = 1,709 thousand riders in Jan 1991 |
| `Fₜ₊ₖ` | Forecast for k periods ahead | F₁₄₈ = forecast for April 2003 |
| `eₜ = Yₜ − Fₜ` | Forecast error | How far off our prediction was |

### Descriptive vs. Predictive — Two Different Goals

- **Time Series Analysis** (descriptive): *"What patterns exist in our Amtrak data? Is ridership growing? Which months are busiest?"* — Understanding the past.
- **Time Series Forecasting** (predictive): *"How many riders will we have in April 2004?"* — Predicting the future.

This course focuses on **forecasting**, but understanding the patterns (analysis) is a necessary first step.

### The 4 Questions Every Forecaster Must Answer

Before touching any data or model, answer these:

**1. What is the forecast horizon?** How far ahead do you need to predict?
   - Are you making a one-time forecast for next quarter, or ongoing monthly forecasts?
   - This determines how much historical data you need and which methods will work
   - Short-term (next day) vs. long-term (next year) forecasts require very different approaches

**2. How will the forecast be used?** Who needs it and what decisions depend on it?
   - Numerical forecast (how many riders?) or binary (will ridership exceed 2 million?)
   - Is over-predicting worse than under-predicting? (e.g., over-ordering inventory vs. stock-out)
   - Will managers "adjust" your numbers based on gut feeling?

**3. What expertise and automation are available?**
   - In-house team or consultants?
   - One series or thousands of products to forecast?
   - What data and software do you have?

**4. Is the data time-series or cross-sectional?** This determines your entire analytical approach.

> **Our Running Example — Amtrak:** Monthly ridership (in thousands) on Amtrak trains from January 1991 to March 2004. We have 159 data points. Goal: forecast the next 36 months. We'll use this dataset all the way through Chapters 5–8.

---

## Chapter 2 — Getting to Know Your Data

### Why This Matters

You've defined the problem (Chapter 1). Now you need data. But not just any data — the right data, in good shape. Just as a chef needs quality ingredients before cooking, a forecaster needs quality data before modeling.

### The 4 Data Collection Considerations

**1. Data Quality** — Garbage in, garbage out. Time series are typically *small* samples (unlike big data), so every data point counts. The data must match exactly what you're forecasting. If you want to forecast total Amtrak ridership, don't use data from just one route.

**2. Temporal Frequency** — How often is data recorded? Every minute? Daily? Monthly? Quarterly?
   - High-frequency data (e.g., every second) captures lots of detail but also lots of **noise**
   - Low-frequency data (e.g., yearly) is smooth but may miss important patterns
   - Sometimes you **aggregate** (e.g., daily → monthly) to balance signal and noise, then **disaggregate** the forecasts back

**3. Series Granularity** — How detailed is the coverage?
   - Geographic: national vs. state vs. city level?
   - Population: all customers vs. segments (seniors, families)?
   - More granular = more noise but potentially more relevant
   - Very granular data may have many zeros → special methods needed

**4. Domain Expertise** — Don't work in a vacuum! Talk to people who understand the business.
   - They know about events that affect the data (strikes, policy changes, promotions)
   - They can validate whether your patterns make sense
   - Affects the entire modeling process from start to end

### The Anatomy of a Time Series — Four Components

Every time series is made up of some combination of these building blocks:

```
Yₜ  =  Level  +  Trend  +  Seasonality  +  Noise
         │          │           │              │
   Baseline     Long-term    Repeating     Random
   value        direction    pattern       variation
```

Let's understand each one deeply:

**1. Level** (always present)
- The "baseline" or "average" value of the series
- Think of it as: if you removed all trends, seasons, and noise, what's left
- Example: Amtrak's base ridership is roughly 1,700 thousand riders per month

**2. Trend** (may or may not be present)
- A steady, long-term increase or decrease
- Can be **linear** (constant growth in absolute terms, e.g., +5 riders/month)
- Can be **exponential** (constant growth in percentage terms, e.g., +2%/month)
- Can be **quadratic** (growth rate itself changes over time)
- Can be **local** (changes direction; sometimes up, sometimes down)
- Example: Amtrak shows a U-shaped (quadratic) trend — declined then recovered

**3. Seasonality** (may or may not be present)
- A pattern that **repeats at a fixed, known period**
- Monthly data might have 12-month seasonality (every January similar, etc.)
- Weekly data might have 7-day seasonality (weekdays vs. weekends)
- Daily data might have 24-hour seasonality (rush hours)
- Some series have **multiple seasonal cycles** (e.g., hourly bikeshare: daily + weekly patterns)
- Example: Amtrak has summer peaks (July/Aug) and winter lows (Jan/Feb) every year

**4. Noise** (always present)
- Random, unpredictable variation
- It's what's left after you account for level, trend, and seasonality
- You **cannot and should not** try to model noise — that leads to **overfitting** (a concept we'll return to again and again)

**Here's how these four components look when separated out visually:**

![Time Series Components](charts/01_ts_components.png)

> **Reading the chart above:** The top-left shows the long-term direction (level + quadratic trend). Top-right shows the 12-month seasonal cycle (summer peaks, winter dips). Bottom-left shows the random noise. Bottom-right combines everything — this is what you actually observe in the data.

> **Real-World Example — Retail Sales:**  
> A clothing store has: **Level** = ~$50,000/month baseline, **Trend** = 3% annual growth, **Seasonality** = December sales 3× higher than February (holiday shopping), **Noise** = random fluctuations due to weather, local events, etc.

### How Components Combine — Additive vs. Multiplicative

The components can combine in two ways:

**Additive:** `Yₜ = Level + Trend + Seasonality + Noise`
- Seasonal swings are roughly **constant** in absolute size over time
- Example: Summer always brings +200 more riders than winter, regardless of the total level

**Multiplicative:** `Yₜ = Level × Trend × Seasonality × Noise`
- Seasonal swings **grow proportionally** with the level
- Example: Summer always brings 20% more riders than winter, so as overall ridership grows, the summer-winter gap widens
- **Trick:** Take log(Yₜ) to convert multiplicative → additive: `log(Yₜ) = log(Level) + log(Trend) + log(Seasonality) + log(Noise)`

![Additive vs Multiplicative Seasonality](charts/02_additive_vs_multiplicative.png)

> **Reading the chart above:** In the left (additive) chart, the seasonal "swing" stays ~40 units whether the series is at 120 or 320. In the right (multiplicative) chart, the swing grows from ~40 to ~120 as the level rises — the swings are proportional to the level.

> **How to tell which you have?** Plot the data. If the seasonal peaks get taller as the series rises → **multiplicative**. If they stay the same height → **additive**.

### Visualizing Your Data — The First Step in Any Analysis

Before running any model, **always plot your data**. Different plots reveal different things:

| Plot Type | What It Reveals | Example Insight |
|-----------|----------------|-----------------|
| **Time plot** | Overall shape, trend, outliers | "Ridership dipped in 1995 then rose" |
| **Trendline overlay** | Direction and shape of trend | "The trend is quadratic (U-shape)" |
| **Seasonal plot** | Seasonal patterns across years | "August is always the peak month" |
| **Interactive (Tableau)** | Zoom, filter, drill-down | "This spike was caused by a holiday" |

### Pre-Processing — Cleaning Before Cooking

Real data is messy. Common issues:
- **Missing values** — "holes" in the series (a month with no data). Solutions: interpolation, or exclude that period
- **Unequally-spaced observations** — data arrives irregularly. May need to regularize
- **Extreme values / outliers** — one month with an absurd spike. Investigate: real event or data error?
- **How far back?** — Using too much history may include irrelevant old patterns; too little may miss trends

> **Transition to Chapter 3:** Now you have clean, explored data. But how will you know if the model you build is actually *good*? You need an evaluation framework *before* building models — that's next.

---

## Chapter 3 — How to Judge a Forecast (Performance Evaluation)

### The Single Most Important Lesson in This Course

> **A model that fits the training data well does NOT necessarily forecast well.**

This deserves a full explanation because it's counterintuitive. Imagine you memorize every answer to a practice exam word-for-word. You'd score 100% on that practice exam. But on the real exam with new questions? You might fail because you memorized instead of understanding.

**Overfitting** is the forecasting equivalent: your model memorizes the training data (including its random noise), and performs poorly on new, unseen data. A perfect fit (like a wiggly line passing through every point) is usually a *terrible* forecaster.

**Underfitting** is the opposite: your model is too simple to capture the real patterns. A flat horizontal line ignores both trend and seasonality.

The sweet spot? **Enough complexity to capture the systematic patterns (trend, seasonality) but not so much that you model the noise.**

![Underfitting vs Good Fit vs Overfitting](charts/04_overfitting.png)

> **Reading the chart above:** Left: the straight line is too simple — it misses the curve (underfitting). Middle: the smooth curve captures the real pattern without chasing noise (good fit). Right: the wiggly line passes through every data point but would give terrible predictions for new points (overfitting).

> **Example — Polynomial Degree and Overfitting:**  
> Fitting a degree-1 polynomial (line) to Amtrak data → underfit (ignores the U-shape).  
> Fitting a degree-2 polynomial (quadratic) → good fit (captures the U-shape).  
> Fitting a degree-20 polynomial → overfit (wiggles through every point, explodes in the test period).

### Data Partitioning — The Foundation of Honest Evaluation

To test whether a model forecasts well, you need to pretend you don't know the future and see how the model does:

```
|←——— Training Period ———→|←—— Test Period ——→|←— Future (unknown) —→
|  Jan 1991 ... Mar 2001   | Apr 2001...Mar 2004| Apr 2004 onwards
|  Fit the model here       | Evaluate here      | Deploy forecasts
```

**Rules:**
1. **Fit the model ONLY on the training period**
2. **Evaluate on the test period** (pretend it's the future)
3. **Deploy:** Combine training + test, re-fit, and forecast the actual unknown future

**Why sequential, NOT random?** Unlike cross-sectional data, time order matters. If you randomly select months for training, you might "train" on Dec 2003 while "testing" on Jan 1993 — that's like seeing the future before predicting the past. It destroys the integrity of evaluation.

**Why no separate validation set (unlike cross-sectional ML)?** Time series datasets are typically small. Splitting into 3 sets leaves too little data. Instead, we use the test period for both tuning and evaluation, but we must be careful.

**Why recombine training + test for deployment?** Because more data = better model. The test period was only "held out" to evaluate — once we've chosen our method, we use ALL available data for the final model.

![Data Partitioning](charts/03_data_partitioning.png)

> **Reading the chart above:** The blue portion is used to estimate model parameters. The red portion is held out — the model has never seen this data. We compare forecasts (from the blue-trained model) against red actuals to measure performance. Only after this evaluation do we combine both for final deployment.

### How to Choose the Test Period

The test period should **mimic your actual forecast horizon**:
- If you need 36-month forecasts → test period should be ~36 months
- If you need next-day forecasts → test period should be recent days
- Match the data frequency

### Measuring How Wrong You Are — Error Metrics

The **forecast error** for each period is:
```
eₜ = Yₜ − Fₜ    (Actual minus Forecast)
```
- Positive eₜ → you **under-predicted** (actual was higher)
- Negative eₜ → you **over-predicted** (actual was lower)

But we need a single number to summarize performance. Here are the main options:

| Metric | Formula | What It Tells You | Weakness |
|--------|---------|-------------------|----------|
| **ME** (Mean Error) | avg(eₜ) | Bias direction (over/under forecast) | +/− cancel out! |
| **MAE** (Mean Abs Error) | avg(\|eₜ\|) | Average magnitude of errors | Scale-dependent |
| **RMSE** (Root Mean Sq Error) | √avg(eₜ²) | Avg magnitude, **penalizes large errors** | Scale-dependent; more pessimistic |
| **MAPE** (Mean Abs % Error) | avg(\|eₜ/Yₜ\|)×100 | Average % off; **scale-independent** | Can't handle Yₜ=0; asymmetric |
| **MASE** (Mean Abs Scaled Error) | see textbook | Compared to naive forecast | Complex formula |

**Deep Dive — Why each matters:**

- **ME** can be zero even if every forecast is wildly wrong (errors cancel: +100, −100 → ME=0). Use it only to detect **bias** (consistent over/under forecasting).
- **MAE** tells you "on average, how many units off are we?" Easy to interpret. For Amtrak: MAE = 150 means "on average, we're off by 150 thousand riders."
- **RMSE** is always ≥ MAE. The squaring means one huge error (e.g., 500) counts much more than many small ones. Use RMSE when **large errors are especially costly**.
- **MAPE** is the go-to for comparing across different series (e.g., comparing accuracy for Amtrak vs. airline data). But it **cannot be computed when actuals are zero**, and it penalizes under-forecasting more than over-forecasting.
- **MASE** < 1 means your model beats naive forecasts. > 1 means the naive would have been better.

> **Worked Example:**
>
> | t | Actual | Forecast | Error | \|Error\| | Error² | \|Error/Actual\| |
> |---|--------|----------|-------|-----------|--------|-----------------|
> | 1 | 100 | 95 | 5 | 5 | 25 | 5.0% |
> | 2 | 110 | 108 | 2 | 2 | 4 | 1.8% |
> | 3 | 90 | 95 | −5 | 5 | 25 | 5.6% |
> | 4 | 120 | 115 | 5 | 5 | 25 | 4.2% |
>
> ME = (5+2−5+5)/4 = **1.75** (slight under-prediction bias)  
> MAE = (5+2+5+5)/4 = **4.25**  
> RMSE = √(79/4) = **4.44**  
> MAPE = (5.0+1.8+5.6+4.2)/4 = **4.14%**

![Error Metrics](charts/19_error_metrics.png)

> **Reading the chart above:** Left: Blue bars are actual, orange are forecasts — you can see they're close but not identical. Right: The error bars show the sign and magnitude. The metrics box summarizes: ME is small (1.5 → low bias), MAE is 4.5 (average error), MAPE is 4% (pretty good!). Note RMSE (5.0) > MAE (4.5) — RMSE always exceeds MAE because it penalizes big errors more.

> **Real-World Example — When metrics disagree:**  
> Model A: errors = [2, 2, 2, 2] → MAE = 2, RMSE = 2 (consistent small errors)  
> Model B: errors = [0, 0, 0, 8] → MAE = 2, RMSE = 4 (one big error)  
> Same MAE! But RMSE reveals that Model B has one dangerous outlier. If big errors are costly (e.g., medicine dosage), prefer Model A despite identical MAE.

### Prediction Intervals — Admitting Uncertainty

A point forecast ("2,100 thousand riders") sounds precise but hides uncertainty. Better: **"95% chance ridership will be between 1,800 and 2,400."**

**Theoretical approach** (assuming normal errors):
```
Prediction Interval = Fₜ₊ₖ ± z × σ
```
Where σ = standard deviation of forecast errors, z = 1.96 for 95%.

**Problem:** Errors are rarely normal! The interval might be too narrow or wide.

**Empirical approach** (works even for non-normal errors — preferred!):
1. Generate one-step-ahead forecasts on the test period using roll-forward
2. Collect the forecast errors
3. Find the 5th and 95th percentiles of errors: e₍₅₎ and e₍₉₅₎
4. 90% Prediction Interval: `[Fₜ₊₁ + e₍₅₎, Fₜ₊₁ + e₍₉₅₎]`

> **Example (Amtrak):** e₍₅₎ = −369.8, e₍₉₅₎ = +138.1 → 90% PI: [Fₜ₊₁ − 369.8, Fₜ₊₁ + 138.1]. Notice it's **asymmetric** — the interval stretches further below the forecast (369.8) than above it (138.1). This means the model's over-predictions (when the forecast is too high) can be much larger than its under-predictions. The error distribution is skewed, not symmetric.
>
> **Why does asymmetry happen?** Because real forecast errors are often non-normal. In this case, certain months (e.g., low-ridership winter months) see much larger over-prediction errors than the typical under-prediction. A symmetric formula (F ± 250) would miss this — the empirical method captures the true shape of the error distribution.

![Prediction Intervals](charts/18_prediction_intervals.png)

> **Reading the chart above:** The blue line is the point forecast, the shaded blue band is the 90% prediction interval. Notice the interval is wider downward than upward (asymmetric) — reflecting that the model's worst errors are over-predictions (where actuals fall far below the forecast). The actual values (dark line) mostly stay within the band.

### Naive Forecasts — Your Minimum Bar

Before building any sophisticated model, establish a **baseline**. If you can't beat the simplest possible method, your model is useless.

**Naive forecast:** Just use the last observed value.
```
Fₜ₊ₖ = Yₜ     ("Tomorrow will be like today")
```

**Seasonal naive:** Use the value from the same season last year.
```
Fₜ₊ₖ = Yₜ₋ₘ₊ₖ   ("This January will be like last January")
```
Where M = number of seasons (e.g., 12 for monthly data).

![Naive Forecasts](charts/05_naive_forecasts.png)

> **Reading the chart above:** The red dashed line (naive) is just a flat line at the last training value — clearly terrible for seasonal data! The green dashed line (seasonal naive) copies last year's pattern — it captures seasonality perfectly but doesn't account for trend changes. Your model should beat BOTH of these to be worthwhile.

> **Example — Which naive to use?**  
> Daily stock prices with no seasonality → use **Naive** (yesterday's closing price)  
> Monthly ice cream sales with strong summer peaks → use **Seasonal Naive** (same month last year)  
> Hourly electricity demand with daily + weekly patterns → use **Seasonal Naive** with appropriate period

### Fixed Origin vs. Rolling Origin Evaluation

**Fixed origin:** One single train/test split. Evaluate all test-period forecasts at once.
- Simple but depends heavily on where you split

**Rolling origin:** Slide the training window forward one period at a time. At each step:
1. Train on all data up to time t
2. Forecast for t+1
3. Move forward: train up to t+1, forecast t+2
4. Repeat

- You get many one-step-ahead forecasts → more robust evaluation
- Like taking multiple practice exams instead of just one

### Forecast Accuracy vs. Profitability

More accurate ≠ always more profitable!
- **Stock trading:** A model that's only right 51% of the time but right during big moves can be very profitable
- **Demand forecasting:** Over-estimating (waste) vs. under-estimating (stockout) may have very different costs
- The real measure should match the **business cost** of errors

> **Transition to Chapter 5:** Now you have a framework: data is explored (Ch 2), evaluation setup is ready (Ch 3). Time to build your first models! We start with the simplest: **smoothing methods** — no math equations, just adaptive averaging.

---

# Part II: The Toolbox — Building Forecasting Models

The next five chapters each present a forecasting approach. They're ordered from **simplest** to **most complex**, and each new method addresses limitations of the previous ones:

```
Smoothing (Ch 5)     → Simple, adaptive, data-driven
    ↓ limitation: no formal model → can't include external info
Regression (Ch 6)    → Formal model for trend + seasonality
    ↓ limitation: residuals may be correlated
AR/ARIMA (Ch 7)      → Model the autocorrelation; add external info
    ↓ limitation: assumes linear relationships
Neural Nets (Ch 8)   → Handle non-linear complexity
    ↓ special case: what if the outcome is yes/no?
Logistic Reg (Ch 9)  → Forecast binary outcomes
```

---

## Chapter 5 — Smoothing Methods: Your First Forecasting Tools

### The Philosophy

Smoothing methods are **data-driven** — they don't assume a specific mathematical form for the data. Instead, they "smooth out" the noise to reveal underlying patterns. Think of them like squinting at a noisy image: you lose the fine detail (noise) but see the big picture (pattern).

They're popular because they're:
- Simple to understand and explain
- Fast and memory-light
- Easy to automate
- No statistical distribution assumptions needed

### Tool #1: Moving Averages

#### The Idea
Forecast by averaging the last W values. The average "smooths out" random ups and downs.

#### Two Types of Windows

**Centered MA** — Window centered around time t (uses past AND future values):
```
Centered MA at t (W=5): average of Yₜ₋₂, Yₜ₋₁, Yₜ, Yₜ₊₁, Yₜ₊₂
```
- Great for **visualization** (shows level and trend clearly)
- **Can't be used for forecasting** because it needs future values!

**Trailing MA** — Window ending at time t (only past values):
```
Trailing MA at t (W=5): average of Yₜ₋₄, Yₜ₋₃, Yₜ₋₂, Yₜ₋₁, Yₜ
```
- **Used for forecasting:** Fₜ₊ₖ = trailing MA at time t

> **Example:** With values Y₃=105, Y₄=115, Y₅=120 and W=3:
> Trailing MA₅ = (105 + 115 + 120)/3 = **113.33**
> The forecast for t=6,7,... is all 113.33 (flat line into the future)

#### Choosing Window Width (W)

This is the key decision:
- **Too narrow** (small W) → not enough smoothing, noise gets through
- **Too wide** (large W) → over-smoothing, masks real patterns, "lags behind" trends
- **For seasonal data:** Set W = number of seasons (e.g., W=12 for monthly) to average out the seasonal cycle
- **No seasonality:** Use a narrow window

![Moving Averages](charts/06_moving_averages.png)

> **Reading the chart above:** The gray line is the noisy original series. The red line (W=3) is barely smoothed — still jagged. The blue line (W=12) nicely reveals the trend by averaging out the 12-month seasonality. The green line (W=24) is over-smoothed — it's so smooth it "lags behind" when the trend changes direction.

> **Example — Choosing W for quarterly data:**  
> Quarterly revenue data has 4 seasons per year.  
> Use W=4 for visualization (averages out the 4 quarters).  
> W=2 would leave seasonal noise. W=8 would over-smooth.

#### Limitation of MA for Forecasting

Moving averages produce a **flat** forecast into the future. They can't capture trend or seasonality! 

**Solution:** Strip away trend and seasonality first (via **differencing**), forecast the remainder, then add the patterns back.

### Differencing — Removing Patterns Before Forecasting

Differencing is a simple operation that removes patterns by subtracting earlier values:

| Type | Formula | What It Removes | Example |
|------|---------|-----------------|---------|
| **Lag-1** | Yₜ − Yₜ₋₁ | Linear trend | "How much did ridership change month-to-month?" |
| **Lag-M** | Yₜ − Yₜ₋ₘ | Seasonality (M periods) | "How does this January compare to last January?" (M=12) |
| **Double** | Difference a differenced series | Both trend + seasonality | First remove seasonality, then remove trend |

**Why differencing is powerful:** Unlike regression-based trend removal (Chapter 6), differencing does NOT assume a global pattern. It handles **local, changing trends** gracefully.

> **Example (monthly data):**
> - Original: 100, 103, 108, 115, 124 → increasing trend
> - Lag-1 differenced: 3, 5, 7, 9 → removed the level, exposed the acceleration
> - If the differences were constant (e.g., all 5), the original had a linear trend
> - If the differences are growing (3, 5, 7, 9), the trend is accelerating → may need another round

![Differencing](charts/07_differencing.png)

> **Reading the chart above:** Left: the original Amtrak-like series has both trend and seasonality. Middle: lag-1 differencing removed the trend — you can see the series now oscillates around zero, but the 12-month seasonal cycle is still visible. Right: lag-12 differencing removed the seasonality — the seasonal pattern is gone, revealing a trend-like pattern.

> **Example — Double differencing for monthly retail sales:**  
> Step 1: Lag-12 difference (remove monthly seasonality): Yₜ − Yₜ₋₁₂  
> Step 2: Lag-1 difference on the result (remove remaining trend): (Yₜ − Yₜ₋₁₂) − (Yₜ₋₁ − Yₜ₋₁₃)  
> Result: a roughly stationary series that can be forecast with SES or ARIMA.

### Tool #2: Simple Exponential Smoothing (SES)

#### The Idea
Like a trailing MA, but instead of giving equal weight to the last W values, give **more weight to recent observations** and exponentially decreasing weight to older ones.

#### The Formula
```
Lₜ = α·Yₜ + (1−α)·Lₜ₋₁
Forecast: Fₜ₊ₖ = Lₜ for all k (flat into the future)
```

Where Lₜ is the estimated "level" and α is the **smoothing constant** (0 < α ≤ 1).

#### Why "Exponential"?
Substitute recursively:
```
Lₜ = α·Yₜ + α(1−α)·Yₜ₋₁ + α(1−α)²·Yₜ₋₂ + α(1−α)³·Yₜ₋₃ + …
```
The weights α, α(1−α), α(1−α)², … decay **exponentially**:

| α | Weight on Yₜ | Yₜ₋₁ | Yₜ₋₂ | Yₜ₋₃ |
|---|---|---|---|---|
| 0.9 | 0.900 | 0.090 | 0.009 | 0.001 |
| 0.5 | 0.500 | 0.250 | 0.125 | 0.063 |
| 0.2 | 0.200 | 0.160 | 0.128 | 0.102 |
| 0.1 | 0.100 | 0.090 | 0.081 | 0.073 |

#### The Adaptive Learning View
Equivalently:
```
Fₜ₊₁ = Fₜ + α·(Yₜ − Fₜ) = Fₜ + α·Eₜ
```
Read this as: *"New forecast = old forecast + correction based on how wrong we were."*
- α controls the **learning rate**: how much we adjust based on the latest error
- Large α → aggressive learning, responsive to recent data, but volatile
- Small α → slow learning, stable forecasts, but slow to react to changes

> **Worked Example:** F₅=150, Y₅=160, α=0.2
> E₅ = 160 − 150 = 10
> F₆ = 150 + 0.2×10 = **152** (adjusted up toward the actual, but cautiously)

#### Choosing α
- Typical values: 0.1, 0.2
- α = 1 → only the last observation matters (same as naive forecast!)
- α → 0 → all history matters equally (extremely smooth, slow to react)
- Relationship to MA: SES with α ≈ trailing MA with W = 2/α − 1
  - Example: α = 0.2 → roughly equivalent to W = 9
- Can optimize by minimizing RMSE/MAPE on training data — but **beware overfitting!**

![Exponential Smoothing](charts/08_exponential_smoothing.png)

> **Reading the chart above:** Left: How weights decay for different α values. With α=0.9 (red), almost all weight is on the most recent observation. With α=0.05 (blue), weights spread far into the past. Right: Applied to a random-looking series — α=0.9 (red) tracks every wiggle, α=0.1 (blue) gives a smooth line that barely reacts.

> **Example — Step-by-step SES calculation (α = 0.3):**  
> Given: Y₁=100, Y₂=110, Y₃=105. Initialize L₁=Y₁=100.
>
> | t | Yₜ | Lₜ = 0.3·Yₜ + 0.7·Lₜ₋₁ | Forecast Fₜ₊₁ |
> |---|-----|------|------|
> | 1 | 100 | 100 | 100 |
> | 2 | 110 | 0.3(110) + 0.7(100) = **103** | 103 |
> | 3 | 105 | 0.3(105) + 0.7(103) = **103.6** | 103.6 |
> | 4 | ? | — | **103.6** (flat into the future) |

#### Limitation
Like MA, SES produces a **flat** forecast. Only works for series with **no trend and no seasonality**. For the full Amtrak data, we'd need to difference first, then apply SES.

### Tool #3: Holt's Method (Double Exponential Smoothing)

#### The Idea
Add a **trend component** to SES. Now we're tracking two things: level AND trend.

#### The Equations
```
Level:    Lₜ = α·Yₜ + (1−α)·(Lₜ₋₁ + Tₜ₋₁)
Trend:    Tₜ = β·(Lₜ − Lₜ₋₁) + (1−β)·Tₜ₋₁
Forecast: Fₜ₊ₖ = Lₜ + k·Tₜ
```

**Key insight:** The trend is **local** — it adapts over time! Unlike regression (Chapter 6) which fits one global trend line, Holt's method lets the trend evolve. This is a major advantage for series where the growth rate changes.

Two smoothing constants: α (for level) and β (for trend). Small β → trend changes slowly (nearly global). Large β → trend reacts quickly.

### Tool #4: Holt-Winter's Method (Triple Exponential Smoothing) ★

#### The Idea
The crown jewel of smoothing: captures **level + trend + seasonality** all at once.

#### The Equations (Multiplicative Seasonality)
```
Forecast: Fₜ₊ₖ = (Lₜ + k·Tₜ) × Sₜ₊ₖ₋ₘ
Level:    Lₜ = α·(Yₜ/Sₜ₋ₘ) + (1−α)·(Lₜ₋₁ + Tₜ₋₁)
Trend:    Tₜ = β·(Lₜ − Lₜ₋₁) + (1−β)·Tₜ₋₁
Season:   Sₜ = γ·(Yₜ/Lₜ) + (1−γ)·Sₜ₋ₘ
```

Three smoothing constants: α, β, γ. M = number of seasons.

**Reading the parameters (from Amtrak example):**
- α far from zero → level adapts locally (responsive)
- β near zero → trend is stable/global (slowly changing)
- γ near zero → seasonality is stable/global (repeats consistently)

#### AutoETS — Automated Selection
Instead of choosing additive vs. multiplicative manually, **AutoETS** tries all combinations:
- Error type: Additive (A) or Multiplicative (M)
- Trend: None (N), Additive (A), Additive Damped (Ad), Multiplicative (M), Mult Damped (Md)
- Seasonality: None (N), Additive (A), Multiplicative (M)

The "damped" variants assume the trend will eventually flatten — useful for long-horizon forecasts where infinite growth is unrealistic.

> **Worked Example — Holt-Winter's one-step forecast:**  
> Suppose at end of training: L₁₂₃ = 1735, T₁₂₃ = 0.9, S₁₁₁ = 1.18 (for month 4, using M=12, multiplicative)
>
> Forecast for period 124 (1 step ahead, month=April):  
> F₁₂₄ = (L₁₂₃ + 1·T₁₂₃) × S₁₁₁ = (1735 + 0.9) × 1.18 = **2,048** thousand riders  
> For period 125 (2 steps ahead, month=May):  
> F₁₂₅ = (1735 + 2×0.9) × S₁₁₂ — using the May seasonal index

![Holt-Winters Decomposition](charts/09_holt_winters.png)

> **Reading the chart above:** Top-left: The raw series with all patterns mixed together. Top-right: The estimated level + trend (the smooth blue curve the series oscillates around). Bottom-left: The seasonal pattern — a repeating wave. Bottom-right: What's left (residuals) after removing level+trend+seasonality — ideally just random noise.

#### Multiple Seasonal Cycles
For complex seasonality (e.g., bikeshare data with both daily and weekly cycles):
1. Use **STL decomposition** (Seasonal-Trend decomposition using Loess) to separate components
2. Forecast the deseasonalized series with ETS or ARIMA
3. Re-seasonalize the forecasts

### Summary: The Smoothing Method Family

```
                    SES (level only)
                     │
              Holt's (+ trend)
                     │
           Holt-Winter's (+ seasonality)    ← Most complete
                     │
              AutoETS (automated ETS)
                     │
           STL + ETS (multiple seasonal cycles)
```

**Key limitation of ALL smoothing methods:** They're purely data-driven. You **cannot include external information** (weather, holidays, economic indicators). For that, you need regression — which is next.

> **Transition:** Smoothing is great for adaptive, automated forecasting. But what if you want to understand *why* the trend or seasonality looks the way it does? Or add external knowledge? Time for **regression models**.

---

## Chapter 6 — Linear Regression: Modeling Trend and Seasonality

### Why Regression After Smoothing?

Smoothing methods treat the series as a black box: data goes in, forecasts come out. Regression gives you a **formal mathematical model** — an equation you can interpret, explain, and extend.

| Aspect | Smoothing | Regression |
|--------|-----------|------------|
| Approach | Data-driven, adaptive | Model-driven, global |
| Interpretability | Limited (just parameters α,β,γ) | High (coefficients have meaning) |
| Trend type | Local (adapts over time) | Global (one equation for all time) |
| External predictors | Cannot include | CAN include |
| Overfitting risk | Moderate | Lower (fewer parameters) |

### Basics of Linear Regression (Quick Refresh)

If this is new to you: regression finds the "best-fit" line (or surface) through data by minimizing the sum of squared errors. The output is:
```
ŷ = β₀ + β₁·x₁ + β₂·x₂ + … + ε
```
- β₀ = intercept (baseline)
- β₁, β₂, ... = coefficients (slope for each predictor)
- ε = error term (what the model can't explain)
- "Best fit" = minimizes Σ(eₜ)² (least squares)

For time series, the trick is: **what are the predictors?** We use the time index, dummy variables, and transformations.

### Model 1: Linear Trend
```
Yₜ = β₀ + β₁·t + ε
```
- β₀ = intercept (estimated starting value)
- β₁ = slope (constant amount of change per period)
- The ONLY predictor is the time index t = 1, 2, 3, …

> **Example (Amtrak):** Ŷ₁₄₈ = 1750.36 + 0.3514×148 = **1,802** thousand riders  
> Actual: 2,099 → Error: **297** (terrible! because trend only, no seasonality)

**Critical Warning:** A significant β₁ does NOT mean a linear trend is adequate. An insignificant β₁ does NOT mean there's no trend!
- The Amtrak trend coefficient (0.35) was insignificant (p=0.39) — but after removing seasonality, the trend becomes significant
- **Best practice:** Always check the deseasonalized data, visualize the trend, and evaluate test-period performance

### Model 2: Exponential Trend
```
log(Yₜ) = β₀ + β₁·t + ε    →    Yₜ = e^(β₀ + β₁·t + ε)
```
- Captures **percentage** growth (constant rate of change)
- Use when growth is proportional to the current level (like compound interest)

### Model 3: Quadratic Trend
```
Yₜ = β₀ + β₁·t + β₂·t² + ε
```
- Captures a **curved** trend (acceleration or deceleration)
- The Amtrak data has a U-shape — ridership dipped then recovered

> **Example (Amtrak quadratic):** Ŷ₁₄₈ = 1888.88 − 6.298×148 + 0.0536×148² = **2,131** (much closer to actual 2,099!)

![Trend Models](charts/10_trend_models.png)

> **Reading the chart above:** All three models are fitted to the deseasonalized Amtrak data (dots). Left: A linear trend (red line) — misses the curvature. Middle: An exponential trend (green curve) — similar to linear over this short horizon. Right: A quadratic trend (blue curve) — captures the U-shape well. The quadratic model would give much better forecasts.

> **Example — Choosing the right trend model:**  
> **Startup revenue:** Doubling each year = exponential trend → use log(Yₜ) = β₀ + β₁t  
> **Population growth:** Slowing growth = quadratic or damped exponential  
> **Global temperature:** Steady +0.02°C/year = linear trend → Yₜ = β₀ + β₁t  
> **Amtrak ridership:** Declined then recovered = quadratic (U-shape) → Yₜ = β₀ + β₁t + β₂t²

### Model 4: Additive Seasonality (Dummy Variables)

Now let's capture the seasonal pattern. The technique: **dummy variables** (also called indicator variables).

For M seasons, create **M−1** dummy variables. Why M−1? One season serves as the "reference" (baseline). Each dummy's coefficient measures the **difference** from that reference.

**For monthly data (M=12), we create 11 dummies:**
```
D_Jan = 1 if month is January, 0 otherwise
D_Feb = 1 if month is February, 0 otherwise
...
D_Nov = 1 if month is November, 0 otherwise
(December = reference: all dummies = 0)
```

**Model:**
```
Yₜ = β₀ + β₁·D_Jan + β₂·D_Feb + … + β₁₁·D_Nov + ε
```
- β₀ = average value in December (reference month)
- β₃ (March coefficient) = how much March is above/below December on average

![Seasonal Dummies](charts/11_seasonal_dummies.png)

> **Reading the chart above:** Each bar shows the seasonal coefficient for that month relative to December (the reference = 0). August has the highest coefficient (+397) — meaning Aug ridership averages 397 thousand MORE than December. February is lowest (−90) — the quietest month. This pattern repeats every year.

> **Example — Dummy variables for quarterly data (M=4):**  
> 3 dummies: D_Q1, D_Q2, D_Q3 (Q4 is reference)  
> For Toys "R" Us: D_Q4 would have the largest positive coefficient (holiday shopping).  
> Model: Yₜ = β₀ + β₁·D_Q1 + β₂·D_Q2 + β₃·D_Q3 + ε  
> Forecast for Q1: Y = β₀ + β₁ (only D_Q1=1, rest=0)

### Model 5: Quadratic Trend + Additive Seasonality ★

The most complete regression model — combines everything:
```
Yₜ = β₀ + β₁·t + β₂·t² + β₃·D_Jan + … + β₁₃·D_Nov + ε
```
Total predictors: 2 (trend: t, t²) + 11 (seasonal dummies) = **13 predictors**

> **Example (Amtrak, April 2003):**  
> F₁₄₈ = 1697 − 7.156(148) + 0.0607(148²) + 260.6 = **2,228** thousand riders  
> (Here 260.6 is the April seasonal coefficient)

### Multiplicative Seasonality
Apply seasonal dummies to **log(Yₜ)**:
```
log(Yₜ) = β₀ + β₁·D_Jan + … + β₁₁·D_Nov + ε
```
Coefficients now represent **percentage** differences between seasons.

### Smooth Seasonality — Fourier Terms
For smoothly-transitioning seasonal patterns (not abrupt jumps between months):
```
Yₜ = β₀ + β₁·t + β₂·t² + β₃·sin(2πt/M) + β₄·cos(2πt/M) + ε
```
Where M = seasonal period (12 for monthly, 365.25 for daily, 52.18 for weekly).

The sine/cosine pair traces a smooth wave over the year. You can add more pairs (higher harmonics) for more complex seasonal shapes.

> **Application (Serfling method in epidemiology):** Used by the CDC to establish a "baseline" of expected pneumonia deaths, against which actual deaths are compared to detect flu outbreaks.

### Key Takeaway — Regression vs. Smoothing

| Feature | Regression (Ch 6) | Smoothing (Ch 5) |
|---------|-------------------|-------------------|
| Trend | Global (linear, quadratic, exponential) | Local (adapts period by period) |
| Seasonality | Fixed pattern (dummy coefficients don't change) | Adaptive (evolves over time) |
| Interpretability | Everything has a coefficient and p-value | α, β, γ only |
| Extrapolation risk | High (quadratic can explode far into the future) | Lower (anchored to recent data) |

**Limitation of Chapter 6 regression:** Look at the residuals (errors) from Model 5 — they're NOT random! There's a pattern: when one month has a positive error, the next month tends to have a positive error too. This is **autocorrelation**, and it means there's information in the residuals we haven't captured yet. That's exactly what Chapter 7 addresses.

> **Transition:** Our regression model captured trend and seasonality but left correlated errors. By modeling these correlations, we can squeeze out even more accuracy — and that's the world of autoregressive models.

---

## Chapter 7 — Autocorrelation, AR Models, ARIMA, and External Information

### What Is Autocorrelation and Why Should You Care?

**Autocorrelation** = correlation of a series with its own past values.

Think of it this way: today's temperature is related to yesterday's temperature. If it was hot yesterday, it's probably hot today. So knowing yesterday's value gives you information about today's — if you can model this, you can improve your forecasts.

**Lag-k autocorrelation** = correlation between (Y₁,…,Yₜ₋ₖ) and (Yₖ₊₁,…,Yₜ)

**The ACF (Autocorrelation Function) Plot:**
- X-axis: lag (1, 2, 3, …, 24)
- Y-axis: autocorrelation at that lag
- Bars beyond the shaded confidence band (±2 standard errors) = significant

**How to read it:**
| ACF Pattern | Meaning |
|-------------|---------|
| Positive lag-1 ("stickiness") | High → high, low → low. The series persists. |
| Negative lag-1 ("swings") | High → low, low → high. The series oscillates. |
| Positive at lags 12, 24, 36… | Monthly seasonality! |
| All bars near zero | The series is essentially random → hard to forecast |

![ACF Plots](charts/12_acf_plots.png)

> **Reading the chart above:** Left: ACF of the raw Amtrak-like series shows strong autocorrelation at all lags, with peaks at lags 12, 24 (seasonality). The red shaded band is the confidence interval — bars exceeding it are significant. Right: ACF of residuals AFTER fitting trend + seasonality — the seasonal pattern is gone, but there's still significant lag-1 autocorrelation. This is what the AR model (Level 2) will capture.

> **Example — What different ACF patterns look like:**  
> **Amtrak raw data:** Slowly decaying positive ACF + peaks at 12, 24 = **trend + seasonality**  
> **Stock price changes:** All bars within confidence band = **random walk** (unpredictable)  
> **Daily temperature:** High lag-1, then slowly decaying = **strong short-term persistence**  
> **Hourly call center volume:** Peaks at lags 24, 48, 72 = **24-hour seasonality**

### Where Autocorrelation Fits in Our Story

After fitting Model 5 (quadratic trend + seasonality) in Chapter 6, the residuals showed strong lag-1 autocorrelation — a pattern the regression didn't capture. If residuals are autocorrelated, it means: *"When the model over-predicts this month, it will probably over-predict next month too."*

This is wasted information! We can model it to improve forecasts.

### Autoregressive (AR) Models

An AR model predicts the current value from its own past values — like a regression, but the predictors are lagged versions of the same series.

**AR(p) Model:**
```
Yₜ = α + β₁·Yₜ₋₁ + β₂·Yₜ₋₂ + … + βₚ·Yₜ₋ₚ + εₜ
```
- p = "order" = how many lags to include
- AR(1): Yₜ depends only on Yₜ₋₁
- AR(2): Yₜ depends on Yₜ₋₁ and Yₜ₋₂

**Forecasting with AR(2):**
```
1-step ahead:  Fₜ₊₁ = a + b₁·Yₜ + b₂·Yₜ₋₁       (uses actual data)
2-step ahead:  Fₜ₊₂ = a + b₁·Fₜ₊₁ + b₂·Yₜ        (uses 1 forecast + 1 actual)
3-step ahead:  Fₜ₊₃ = a + b₁·Fₜ₊₂ + b₂·Fₜ₊₁      (uses 2 forecasts)
```
Notice: the further ahead you go, the more you rely on forecasts rather than actuals. **This is why AR models are most useful for short-term forecasting.**

### The Two-Level Model — A Powerful Strategy ★

This is one of the most practical techniques in the course:

```
Level 1: Fit trend + seasonality model → get forecasts F and residuals E
Level 2: Fit AR model to residuals → get forecasted errors Ê
Combined: F* = F + Ê (improved forecast)
```

**Step-by-step (Amtrak example):**

1. **Level 1:** Model 5 (quadratic + seasonal) gives F₁₄₈ = 2,004.27 for April 2001
2. **Check residuals:** ACF shows significant lag-1 autocorrelation
3. **Level 2:** Fit AR(1) to residuals: Êₜ = 0.1492 + 0.5998·Eₜ₋₁
4. **Most recent residual** (March 2001): E = 12.108
5. **Forecasted error:** Ê = 0.1492 + 0.5998×12.108 = **7.411**
   - Interpretation: the model tends to under-predict by about 7.4, so adjust UP
6. **Combined forecast:** F* = 2,004.27 + 7.41 = **2,011.68** (Actual: 2,023.79)

This is better than the Level 1 forecast alone!

**When does it help?** Only for short-term horizons. For long horizons, the AR correction degrades because it feeds on its own forecasts.

![Two-Level Model](charts/13_two_level_model.png)

> **Reading the chart above:** Top panel: The blue line (Level 1) captures the overall shape but doesn't perfectly track the actual (black). Middle panel: The residuals show autocorrelation — runs of positive and negative values. Bottom panel: The green line (Level 1 + AR correction) tracks the actual much more closely than the blue line alone. The AR layer "fine-tunes" the forecast.

> **Example — When the two-level model helps vs. doesn’t:**
> - **Helps:** Forecasting next month's Amtrak ridership (short-term, with autocorrelated residuals)  
> - **Doesn't help:** Forecasting ridership 24 months ahead (AR forecasts degrade over many steps)  
> - **Helps:** Daily electricity demand (strong day-to-day persistence in residuals)  
> - **Doesn't help:** When residuals are already random (no autocorrelation to model)

### Evaluating Predictability — The Random Walk Test

A crucial question: *"Is this series even predictable, or are changes purely random?"*

**Random Walk = AR(1) with β₁ = 1:**
```
Yₜ = a + Yₜ₋₁ + εₜ
```
Meaning: the best forecast for tomorrow is **today's value** plus a drift. You can't beat the naive forecast.

**How to test:**
- Fit AR(1): Yₜ = α + β₁·Yₜ₋₁ + ε
- Test H₀: β₁ = 1
- Test statistic: |β₁ − 1| / SE(β₁)
- If > 1.96 at 5% level → reject → NOT a random walk → **proceed with forecasting**
- If ≈ 0 → cannot reject → may be random walk → **naive forecast is optimal**

> **Example — S&P 500:** β₁ = 0.9841, SE = 0.014. Test: |0.9841−1|/0.014 = 1.14 < 1.96. Cannot reject → **random walk** → stock prices are unpredictable! → **Efficient Market Hypothesis:** prices already reflect all information.
>
> **Example — Amtrak residuals:** β₁ = 0.5998, SE = 0.0712. Test: |0.5998−1|/0.0712 = 5.62 >> 1.96. Reject → **NOT random walk** → modeling residuals is worthwhile!

![Random Walk](charts/14_random_walk.png)

> **Reading the chart above:** Left: A random walk looks like real stock prices — it trends up and down, appearing to have patterns. But it’s pure randomness! Right: When you difference it (Yₜ − Yₜ₋₁), the changes are clearly random — no pattern, no autocorrelation. This is the hallmark of a random walk.

> **Real-World Example — Why the S&P 500 is a random walk:**  
> If the S&P 500 were predictable, everyone would trade on the predictions. The trading would push prices to the "correct" value immediately, eliminating the pattern. This self-correcting mechanism is the **Efficient Market Hypothesis** — stock prices already reflect all public information.

### From AR to ARMA to ARIMA

Each layer adds complexity to handle more types of data:

**AR(p):** Uses past values as predictors
```
Yₜ = α + β₁Yₜ₋₁ + … + βₚYₜ₋ₚ + εₜ
```

**ARMA(p,q):** Adds "moving average" terms (lags of forecast ERRORS, not the series itself)
```
Yₜ = α + β₁Yₜ₋₁ + … + βₚYₜ₋ₚ + εₜ − θ₁εₜ₋₁ − … − θqεₜ₋q
```
- MA(q) terms capture short-lived shocks that affect multiple periods
- **Warning:** "Moving Average" in ARMA has NOTHING to do with the moving average from Chapter 5!

**ARIMA(p,d,q):** Adds differencing to handle non-stationary data (trend/seasonality)
```
ARIMA(p,d,q) = fit ARMA(p,q) to the d-times differenced series
```
- d=0: no differencing
- d=1: one round of lag-1 differencing (removes linear trend)
- d=2: two rounds (removes quadratic trend)

**Seasonal ARIMA:** ARIMA(p,d,q)(P,D,Q)[m]
- Uppercase P,D,Q = seasonal AR, differencing, MA terms
- m = seasonal period (12 for monthly, 52 for weekly)

> **Example (Walmart weekly sales):**
> - Stepwise search: ARIMA(1,0,0)(0,1,0)[52] → seasonal differencing at lag 52 + AR(1)
> - Broad search: ARIMA(0,0,2)(0,1,0)[52] → seasonal differencing at lag 52 + MA(2)
> - Both give MAPE ≈ 15.8% on test — similar performance despite different structures

**ARIMA in Practice:**
- Requires stationarity (no trend/seasonality) → differencing handles this
- More volatile and harder to explain to management
- Automated software (auto ARIMA) searches for best p,d,q using AIC criterion
- Need to show a big advantage over simpler methods to justify the complexity

### Including External Information — The Bridge to the Real World

So far, all methods have been **extrapolation models** — they forecast using only the series' own past history. But real-world forecasts often benefit from **external predictors**: weather, holidays, promotions, economic indicators.

**Only certain methods can include external predictors:** Linear regression (Ch 6), logistic regression (Ch 9), neural networks (Ch 8), and ARIMA with regressors.

**Two critical issues:**

**Issue 1: Must the predictor be lagged?**
```
airfareₜ = β₀ + β₁·gaspriceₜ₋₁ + ε    ← Lagged: gas price from LAST month (available now!) ✓
airfareₜ = β₀ + β₁·gaspriceₜ + ε       ← Concurrent: THIS month's gas price → must forecast it first! ✗
```
**Rule:** If the predictor is concurrent (same time), you need to forecast it too — which adds uncertainty. Use **lagged** predictors whenever possible because they're already known at prediction time.

**Issue 2: Data availability at prediction time**
- Even lagged data may have reporting delays (e.g., GDP is published months after the quarter)
- Always ask: *"Will this predictor actually be available when I need to make my forecast?"*

> **Example (Bike Rentals):** ARIMA + weather forecast + working day indicator. The model captures dips on rainy non-working days but misses some extreme dips → need more external info.
>
> **Example (Walmart):** Adding IsHoliday actually made forecasts WORSE compared to just seasonal ARIMA! Weekly seasonality was more informative than the holiday flag. Lesson: **more predictors ≠ better forecasts**.

> **Transition:** Regression and ARIMA assume linear relationships. But what if the relationship between inputs and output is nonlinear and complex? Enter neural networks.

---

## Chapter 8 — Neural Networks: When Linearity Isn't Enough

### Where This Fits in the Story

So far:
- Smoothing (Ch 5) → adaptive but no external info
- Regression (Ch 6) → interpretable but global and linear
- ARIMA (Ch 7) → handles autocorrelation but still linear

Real-world patterns can be **nonlinear**. The relationship between temperature and ice cream sales isn't a straight line — sales might be flat in cold weather, then spike exponentially once it gets warm enough. Neural networks can capture such patterns.

### The Core Idea

A neural network captures complex relationships by creating **layers of derived variables**. Instead of predicting the output directly from the inputs, it first transforms them through intermediate "hidden" layers.

```
Input Layer  →  Hidden Layer(s)  →  Output Layer
(raw data)      (learned features)    (forecast)
```

Each **node** in the hidden layer:
1. Takes a weighted sum of all inputs + a bias
2. Applies an **activation function** to "squash" or transform the sum
3. Passes the result to the next layer

### Activation Functions — Why They Matter

The activation function is what gives neural networks their power. Without it, stacking layers would just be another linear model!

| Function | Shape | Output Range | Used For |
|----------|-------|-------------|----------|
| **Linear** | Straight line | (−∞, +∞) | Numeric output (same as regression!) |
| **Exponential** | Exponential curve | (0, +∞) | Log-linear models |
| **Logistic (Sigmoid)** | S-shaped | (0, 1) | Binary classification (same as logistic reg!) |
| **Tanh** | S-shaped | (−1, +1) | Common hidden layer activation |
| **ReLU** | Kinked line: max(0,x) | [0, +∞) | Most popular in modern deep learning |

![Activation Functions](charts/16_activation_functions.png)

> **Reading the chart above:** Each panel shows how the function transforms its input. **Linear** passes values through unchanged (just like regression). **Logistic** squashes everything into (0,1) — perfect for probabilities. **Tanh** maps to (−1,+1) — centered at zero. **ReLU** is zero for negative inputs, linear for positive — simple and computationally fast.

> **Example — Why activation matters:**  
> Predicting Amtrak ridership (continuous, positive values) → use **Linear** in output layer  
> Predicting rain/no-rain (binary) → use **Logistic** in output layer  
> Hidden layers: **ReLU** or **Tanh** introduce the nonlinearity that makes NNs powerful

### The Key Insight: NNs Generalize Other Models ★

This is the "aha moment" of the chapter:

| Neural Network Config | Equivalent To |
|----------------------|---------------|
| No hidden layers + linear activation | **Linear regression** (Ch 6)! |
| No hidden layers + exponential activation | **Exponential trend model** (Ch 6)! |
| No hidden layers + logistic activation | **Logistic regression** (Ch 9)! |
| No hidden layers + linear + lag inputs | **AR(p) model** (Ch 7)! |
| WITH hidden layers + nonlinear activation | Something NEW — captures nonlinearity! |

So neural networks are a **superset** that includes the previous methods as special cases. The hidden layers + nonlinear activations are what give them additional power.

### Architecture Decisions

**Number of Hidden Layers:** Usually 1-2 in practice. More → more overfitting risk.

**Number of Nodes:** More nodes = more complexity = more overfitting risk. Start small.

**Choice of Predictors** for time series forecasting:
- Lagged values: Yₜ₋₁, Yₜ₋₂, …, Yₜ₋ₚ
- Seasonal dummies: D_Jan, D_Feb, …
- Time index for trend: t
- External predictors: weather, holidays, etc.

![Neural Network Architecture](charts/15_neural_network.png)

> **Reading the chart above:** This shows a typical NN for time series forecasting. **Input layer** (blue): receives lagged values (Yₜ₋₁, Yₜ₋₂, Yₜ₋₃), a seasonal indicator, and the time index. **Hidden layer** (orange, sigmoid activation): 3 nodes that learn nonlinear combinations of inputs. **Output** (green, linear): produces the forecast Ŷₜ. Each line represents a learned weight.

> **Example — NNetAR for Amtrak:**  
> With p=2 (nonseasonal lags) and P=1 (seasonal lag), input nodes receive: Yₜ₋₁, Yₜ₋₂, Yₜ₋₁₂  
> Hidden layer has (2+1+1)/2 = 2 nodes (rounded)  
> 20 networks are fitted with different random initializations, then averaged  
> This ensemble reduces the risk of getting stuck in a bad local minimum

### How NNs Learn — Backpropagation

The network starts with random weights and iteratively improves:

1. **Forward pass:** Feed data through, get a prediction
2. **Compute error:** How far off is the prediction?
3. **Backward pass (backpropagation):** Trace the error back through the network, adjusting each weight to reduce the error
4. **Repeat** for many iterations (epochs)

**Two updating strategies:**
- **Case updating:** Update weights after each training example
- **Batch updating:** Accumulate errors from all examples, then update

**Learning rate** (0 to 1): How big are the weight adjustments?
- Too large → overshoots, unstable
- Too small → very slow convergence

**Momentum:** Keeps weights from oscillating; if the adjustment was going in a consistent direction, keep going.

### The #1 Danger: Overfitting ★

With enough nodes and iterations, a NN can memorize the training data perfectly — including all the noise. It will then fail spectacularly on new data.

**Four defenses against overfitting:**
1. **Track validation error:** Stop training when test/validation error starts increasing
2. **Limit iterations:** Don't run for thousands of epochs
3. **Limit complexity:** Fewer hidden layers, fewer nodes
4. **Cross-validation:** Evaluate on multiple data splits

### NNetAR — Simplified NN for Time Series

Rather than configuring everything manually, **NNetAR** (Neural Network AutoRegression) is purpose-built for time series:
- Single hidden layer only
- Only uses lag values as predictors (automated p and P selection)
- Logistic activation (input→hidden), Linear activation (hidden→output)
- Fits multiple networks (default: 20) and averages their predictions (ensemble)

**Key parameters:**
- `p` = non-seasonal lags, `P` = seasonal lags
- `period` = seasonal period (e.g., 12 for monthly)
- `n_nodes` = hidden layer size (default: (p+P+1)/2)
- `scale_inputs` = normalize? (Scaling significantly affects performance!)
- `auto` = let the algorithm choose p and P

> **Amtrak Example:** NNetAR results were **inferior to Holt-Winter's** (Chapter 5). Key findings:
> - Scaling inputs had a huge effect on results
> - Fewer lags (p=2, P=1) outperformed many lags (p=11, P=1) — less overfitting
> - More complexity ≠ better forecasts!

### Deep Learning — LSTM Networks

For very long sequences, simple RNNs suffer from the **vanishing gradient problem** — the influence of early time steps "vanishes" as the network gets deeper.

**Long Short-Term Memory (LSTM)** solves this with **gates** that control information flow:
- **Forget gate:** What old information to discard
- **Input gate:** What new information to store
- **Output gate:** What to output

**Setup:** Partition the training series into rolling windows (sub-series) → each window is one training example.

> NNs give mixed results vs. traditional methods. They seem to work best with **high-frequency, large-scale data** where patterns are complex and nonlinear. For the relatively simple Amtrak monthly data, Holt-Winter's was better.

> **Transition:** All methods so far produce **numerical** forecasts. But what if the question is: "Will it rain tomorrow?" or "Will sales go up or down?" That's a **binary** outcome — and requires a different approach.

---

## Chapter 9 — Binary Forecasting and Logistic Regression

### Where This Fits

Sometimes you don't need to know "how much" but "yes or no":

| Scenario | Series Type | Forecast |
|----------|------------|----------|
| "Will ridership go up or down?" | Numerical series | Binary (up/down) |
| "Will it rain tomorrow?" | Binary series | Binary (rain/no-rain) |
| "Will sales exceed $1M?" | Numerical series | Binary (above/below threshold) |

### The Challenge of Binary Data

Binary time series are tricky to visualize. A time plot of 0s and 1s is not informative! Instead:
- **Temporal aggregation:** Plot % of events per month (e.g., % rainy days) across years
- **Polynomial trendlines** on aggregated data to see seasonal patterns

### Naive Benchmarks for Binary Data

Before building models, establish baselines:
1. **Previous value:** "If it rained today, predict rain tomorrow"
2. **Majority vote:** "The most common outcome in training is the forecast"

> **Melbourne example:** Training data (2000-2009): 35.5% rainy, 64.5% no-rain. Majority vote = "no rain" for every single day. This would be right 64.5% of the time — can we beat this?

### The Confusion Matrix — Evaluating Binary Forecasts ★

Instead of MAE/RMSE, binary forecasts use a **classification (confusion) matrix:**

|  | Predicted: Event (Rain) | Predicted: No Event |
|--|:-:|:-:|
| **Actual: Event** | True Positive (TP) | False Negative (FN) — "Missed it!" |
| **Actual: No Event** | False Positive (FP) — "False alarm!" | True Negative (TN) |

**Key metrics:**
- **Overall Accuracy** = (TP + TN) / Total — but can be misleading for imbalanced data!
- **Sensitivity (Recall)** = TP / (TP + FN) — "Of all rainy days, how many did we catch?"
- **Specificity** = TN / (TN + FP) — "Of all dry days, how many did we correctly call dry?"

**Sensitivity vs. Specificity is a trade-off.** Aggressively predicting "rain" catches more rainy days (high sensitivity) but creates more false alarms (low specificity), and vice versa.

### Logistic Regression — The Model

We can't just use linear regression for binary outcomes — it would predict values outside [0,1], which makes no sense for probabilities.

**The solution: the logit transformation.**

Step 1 — **Odds**: If P(rain) = 0.8, the odds = P/(1-P) = 0.8/0.2 = 4 ("4 to 1 in favor of rain")

Step 2 — **Log-odds (logit)**: logit(P) = log(P/(1-P)) — maps [0,1] to (-∞, +∞)

Step 3 — **Model the log-odds with a linear equation:**
```
logit(P(Rainₜ = 1)) = β₀ + β₁·Rainₜ₋₁ + β₂·sin(2πt/365.25) + β₃·cos(2πt/365.25) + ε
```

**To get back to probabilities:**
```
P(Rainₜ = 1) = 1 / (1 + e^(−(β₀ + β₁·Rainₜ₋₁ + β₂·sin(…) + β₃·cos(…))))
```

**Estimation:** Maximum Likelihood Estimation (MLE), NOT least squares. MLE finds the parameter values that make the observed data most likely.

### From Probabilities to Predictions — The Cutoff

Logistic regression outputs **probabilities**. To get binary predictions, choose a **cutoff**:
```
If P(Rain) > cutoff → predict "Rain"
If P(Rain) ≤ cutoff → predict "No Rain"
```

**Default cutoff = 0.5**, but you can adjust:
- Lower cutoff (e.g., 0.3) → predict rain more often → higher sensitivity, lower specificity
- Higher cutoff (e.g., 0.7) → predict rain less often → lower sensitivity, higher specificity

**Choosing the cutoff depends on costs:** If a missed rainy day (FN) is much worse than a false alarm (FP), lower the cutoff to catch more rain.

![Logistic Regression](charts/17_logistic_regression.png)

> **Reading the chart above:** Left: The S-shaped logistic curve maps any input value to a probability between 0 and 1. The red horizontal line at 0.5 is the default cutoff. Values above (orange region) = predict "Rain", below (blue region) = predict "No Rain". Right: A confusion matrix example with TP=120 rainy days caught, FN=40 rainy days missed, FP=30 false alarms, TN=110 correct non-rain. Sensitivity = 75%, Specificity = 79%.

> **Example — Impact of changing the cutoff:**  
> **Cutoff = 0.5:** Sensitivity = 75%, Specificity = 79% (balanced)  
> **Cutoff = 0.3:** Sensitivity = 90%, Specificity = 55% (catch more rain, but many false alarms)  
> **Cutoff = 0.7:** Sensitivity = 50%, Specificity = 92% (miss half the rain, but very few false alarms)  
> 
> For a weather app: lowering cutoff is better (people prefer a false "bring umbrella" over getting soaked)  
> For expensive flood preparation: raising cutoff saves money on false alarms

### What Predictors Can You Use?

Exactly the same types as linear regression:
- **Lagged values** (yesterday's rain)
- **Lagged numerical values** (yesterday's rainfall amount)
- **Trend** (time index)
- **Seasonality** (sine/cosine or dummies)
- **External predictors** (barometric pressure, humidity)
- **Interaction terms**

### Neural Nets for Binary Outcomes

Use `MLPClassifier` from scikit-learn instead of `MLPRegressor`. The output layer uses a logistic activation. In the Melbourne example, NN performance was very similar to logistic regression — for binary time series with limited data, the simpler logistic model is usually sufficient.

### Connection to Everything Else

Logistic regression is to **binary** outcomes what linear regression (Ch 6) is to **numerical** outcomes. The same structure applies:
1. Estimate model on training period
2. Evaluate on test period (using confusion matrix instead of RMSE/MAPE)
3. Re-run on full data for deployment

> **Transition:** You've built models and evaluated them. Now the hardest part: convincing your boss and keeping the system running. That's Chapter 10.

---

# Part III: Deployment

---

## Chapter 10 — Presenting, Monitoring, and Maintaining Forecasts

### The Real World

Your model is only useful if people **trust it, understand it, and use it correctly**. This final chapter addresses the human side of forecasting.

### Presenting Forecasts — The Checklist

Every forecast presentation should include:

| Element | Why It Matters |
|---------|---------------|
| **The forecast itself** | Obviously! With appropriate scale and rounding |
| **Benchmark comparison** | "Our model has MAPE of 8% vs. 15% for naive" |
| **Method used** | Brief, non-technical description for management |
| **Uncertainty** | Prediction intervals! "95% chance between A and B" |
| **Key factors** | "Weather and holidays are the main drivers" |
| **Performance metrics** | Training AND test period accuracy |
| **Scenario analysis** | "What-if" — how forecasts change under different assumptions |

### Monitoring — Is the Model Still Working?

A deployed model assumes "underlying conditions remain unchanged." But the world changes! Use **control charts** for forecast errors:
- Plot errors over time
- Set upper and lower control limits (e.g., ±3 standard deviations)
- If errors consistently fall outside limits → the model needs updating
- Example: Coca-Cola quarterly sales model — when a new product launch changed the pattern, errors started exceeding control limits

### Written Reports

Two-part structure:
1. **Executive Summary:** Problem, data, method, forecasts, precision (non-technical)
2. **Technical Details:** Pre-processing, data sources, method selection rationale, performance evaluation, benchmarks

### Keeping Records — Building Trust Over Time

Document everything:
- Previous forecasts vs. actuals → shows track record
- Error distributions → enables prediction intervals
- Model versions → enables improvement tracking

### The #1 Enemy: Managerial "Adjustments" ★

In practice, managers frequently "adjust" statistical forecasts based on judgment. Research shows this usually **reduces accuracy** because adjustments are:
- Strategic (accounting under-predicts to avoid stockouts; marketing over-predicts for budget)
- Political (compromise between departments)
- Ego-driven (the forecaster wants to show they "add value")

**What to do:**
1. Make the forecasting method transparent and well-documented
2. Keep records of both adjusted and unadjusted forecasts
3. Show comparative performance — let the data speak

### Method Comparison — The Final Picture

![Method Comparison](charts/20_method_comparison.png)

> **Reading the chart above:** Blue bars = training MAPE, red bars = test MAPE. Notice the NNetAR model (far right) has the **lowest training MAPE (3%)** but relatively high test MAPE (11%) — classic **overfitting!** Holt-Winter's achieves a good balance (5% training, 7% test). Naive and SES are the worst on both sets. The best method isn't the one with the lowest training error — it's the one where training and test errors are both low AND close to each other.

> **Key Takeaway:** This chart perfectly illustrates the core lesson of the entire course. Goodness of fit (training error) ≠ forecast accuracy (test error). Always evaluate on the test period.

---

# The Complete Method Selection Guide

Now that you've seen all the tools, here's how to choose:

```
START: Do you have a time series?
  │
  ├─ Is the outcome binary (yes/no)?
  │   └─ YES → Logistic Regression (Ch 9) or NN Classifier (Ch 8)
  │
  ├─ Is the series a random walk? (test with AR(1), Ch 7)
  │   └─ YES → Use Naive forecast. Don't waste time modeling.
  │
  ├─ Need interpretable model with external predictors?
  │   └─ YES → Linear Regression (Ch 6) + AR layer if residuals autocorrelated (Ch 7)
  │
  ├─ Need automated, adaptive forecasting for many series?
  │   └─ YES → Holt-Winter's / AutoETS (Ch 5)
  │
  ├─ Have complex nonlinear patterns + lots of data?
  │   └─ YES → Neural Networks (Ch 8)
  │
  └─ Dealing with autocorrelation + seasonality + external info?
      └─ ARIMA with regressors (Ch 7)
```

### Quick Comparison Table

| Method | Trend | Season | External | Interpret | Automate | Best For |
|--------|:---:|:---:|:---:|:---:|:---:|------|
| Naive | - | ✓(seasonal) | - | ✓✓✓ | ✓✓✓ | Benchmark |
| Moving Average | - | - | - | ✓✓✓ | ✓✓✓ | Visualization |
| SES | - | - | - | ✓✓ | ✓✓✓ | Level-only |
| Holt's | ✓ | - | - | ✓✓ | ✓✓ | Trend, no season |
| Holt-Winter's | ✓ | ✓ | - | ✓✓ | ✓✓ | Trend + season |
| Lin Regression | ✓(global) | ✓ | ✓ | ✓✓✓ | ✓ | Analysis + forecast |
| ARIMA | ✓ | ✓ | ✓ | ✓ | ✓ | Autocorrelated data |
| Neural Nets | ✓ | ✓ | ✓ | ✗ | ✗ | Complex nonlinear |
| Logistic Reg | ✓ | ✓ | ✓ | ✓✓✓ | ✓ | Binary outcomes |

---

# Key Formulas — All in One Place

### Error Metrics
```
eₜ = Yₜ − Fₜ              ME  = avg(eₜ)
MAE = avg(|eₜ|)            RMSE = √avg(eₜ²)
MAPE = avg(|eₜ/Yₜ|) × 100%
```

### Smoothing
```
SES:   Lₜ = α·Yₜ + (1−α)·Lₜ₋₁              Forecast: Fₜ₊ₖ = Lₜ
Holt:  Lₜ = α·Yₜ + (1−α)·(Lₜ₋₁ + Tₜ₋₁)    Forecast: Fₜ₊ₖ = Lₜ + k·Tₜ
       Tₜ = β·(Lₜ − Lₜ₋₁) + (1−β)·Tₜ₋₁
HW:    Fₜ₊ₖ = (Lₜ + k·Tₜ) × Sₜ₊ₖ₋ₘ        (multiplicative)
```

### Regression
```
Linear:      Yₜ = β₀ + β₁·t + ε
Exponential: log(Yₜ) = β₀ + β₁·t + ε
Quadratic:   Yₜ = β₀ + β₁·t + β₂·t² + ε
Seasonal:    Yₜ = β₀ + Σβᵢ·Dᵢ + ε            (M−1 dummies)
Full model:  Yₜ = β₀ + β₁·t + β₂·t² + Σβᵢ·Dᵢ + ε
```

### Autocorrelation & AR
```
AR(1):         Yₜ = α + β₁·Yₜ₋₁ + εₜ
Two-level:     F* = F + Ê
Random walk:   Fₜ₊ₖ = k·a + Yₜ    (if a=0: Fₜ₊ₖ = Yₜ)
RW test:       |β₁ − 1| / SE(β₁) > 1.96 → reject random walk
ARIMA(p,d,q):  ARMA(p,q) on d-times differenced series
```

### Binary Forecasting
```
logit(P) = log(P/(1−P)) = β₀ + β₁·X₁ + β₂·X₂ + ε
P = 1 / (1 + e^(−(β₀ + β₁·X₁ + ...)))
Sensitivity = TP / (TP + FN)
Specificity = TN / (TN + FP)
```

---

*Now go back and trace the story: Define problem → Explore data → Set up evaluation → Build models (simple → complex) → Deploy. Each chapter solves a limitation of the previous one. Good luck tomorrow!* 🎓
