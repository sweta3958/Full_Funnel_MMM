# Does a standard MMM over-credit paid search?

A check on 27 real ecommerce brands, in plain Python.

Paid search mostly collects demand that something else created. A standard marketing mix model (MMM) ignores that, so it tends to give paid search too much credit and the channels that create the demand too little. This repo tests that claim on public data, using a search-demand control and a simple two-stage model.

Companion article: [add your Medium link here]

## The idea

![Causal diagram](charts/causal_diagram.png)

Someone sees a social ad, gets curious and searches. Some of those searchers click a paid search ad. Many would have clicked the free result and bought anyway. A model that ignores this goes wrong in two ways:

1. It over-credits paid search, by counting buyers who were already on their way.
2. It under-credits paid social, by counting only the sales that skipped search.

## Results

**One brand, step by step.** A UK home and garden retailer with 234 weeks of data. Figures are revenue per 1 of spend.

| Model | Paid search | Paid social |
| --- | --- | --- |
| 1. Single-stage (the usual MMM) | 9.04 | 1.04 |
| 2. Search demand added as a control | 6.89 | 0.77 |
| 3. Two-stage | 6.89 | 1.78 |

![One brand, three models](charts/brand_three_models.png)

Search loses about a quarter of its return once search demand is controlled. Paid social gains about 70% once the searches it creates are counted. Search still returns more per pound; the gap narrows from about 9 to 1 down to about 4 to 1.

**All 27 eligible brands.**

| Check | Result |
| --- | --- |
| Search return falls once search demand is controlled | 19 of the 22 brands with a positive search estimate (median -28%) |
| Paid social clearly lifts search demand | 7 of 27 |
| Paid social clearly lowers it | 2 of 27 |
| Too noisy to say | 18 of 27 |
| Two-stage social total is above the single-stage number | 13 of 27 |

![All brands](charts/all_brands_search_change.png)

The first problem is the dependable one: a standard MMM over-credits paid search in most of these brands. The second, social feeding search, is real for some brands and undetectable for most.

## Reproduce it

```bash
pip install -r requirements.txt
python download_data.py     # fetches the dataset from figshare into data/
python analysis.py          # prints the results and writes the charts
```

Tested with Python 3.12, pandas 3.0, numpy 2.4, statsmodels 0.15 and matplotlib 3.10.

If the download script fails, download the files by hand from the [dataset page](https://doi.org/10.6084/m9.figshare.25314841) and put them in `data/`.

## What is in the repo

| File | Purpose |
| --- | --- |
| `download_data.py` | Downloads the dataset into `data/` (not committed) |
| `analysis.py` | Builds weekly data, fits the three models, runs all brands, saves charts |
| `make_diagram.py` | Draws the causal diagram |
| `charts/` | Output charts used in this README |
| `requirements.txt` | Python packages |

## Method in brief

- **Weekly data per brand.** Revenue is purchases at original price minus discounts.
- **Paid social** is Meta impressions with a carry-over (adstock) rate of 0.5.
- **Paid search** is Google paid search clicks.
- **Search demand** is organic search clicks. The dataset has no query volume, so this is a stand-in.
- **Controls** are other paid spend, a linear trend and time of year.
- **Step 1** regresses revenue on social, search and controls.
- **Step 2** adds search demand.
- **Step 3** regresses search demand and paid search clicks on social, then adds those indirect paths to social's direct effect.
- **Brand selection.** The walk-through brand is the longest series among brands where paid social is over half of paid spend. Brands in the cross-brand run need at least 104 weeks, with paid social, paid search and organic search active in most weeks.
- **Standard errors** allow for week-to-week correlation (HAC, 4 lags).

## Limits

- **There is no answer key.** Real data cannot say which model is right. Only an experiment can.
- **Organic clicks are a stand-in.** Paid ads take some clicks that would have gone to the organic result. True query volume is the better control.
- **Levels move with modelling choices.** Using yearly dummies in place of the trend changes search from 9.04 to 5.24 for the walk-through brand. The fall after the control stays at about a quarter.
- **Left out:** saturation curves, an estimated carry-over rate, and impression share.

## Data

[Multi-Region Marketing Mix Modeling (MMM) Dataset for Several eCommerce Brands](https://doi.org/10.6084/m9.figshare.25314841), published on figshare. It is not included in this repo. Please cite it as its page asks.

## Licence

Code is released under the MIT Licence. The dataset keeps its own licence.
