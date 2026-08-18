# PRODIGY_ML_02
Create a K-means clustering algorithm to group customers of a retail store based on their purchase history.

# Mall Customer Segmentation with K-Means

Segments the 200 customers in `Mall_Customers.csv` into behavioral groups
using K-Means clustering, so the store can target marketing, promotions,
and loyalty programs by segment instead of treating everyone the same.

## Dataset

`Mall_Customers.csv` — 200 rows, one per customer:

| Column | Description |
|---|---|
| `CustomerID` | Unique customer identifier |
| `Gender` | Male / Female |
| `Age` | Customer age in years |
| `Annual Income (k$)` | Annual income, in thousands of dollars |
| `Spending Score (1-100)` | Score assigned by the mall based on customer behavior and purchase data (higher = spends more freely) |

## Method

1. **Feature selection** — `Age`, `Annual Income`, and `Spending Score` are
   used as clustering features. `Gender` and `CustomerID` are kept in the
   output but excluded from the model (identifiers and categorical labels
   don't carry distance information K-means can use directly).
2. **Scaling** — All features are standardized (`StandardScaler`) so that
   income (scale of tens of thousands) doesn't dominate the distance
   calculation over spending score (scale of 1–100) or age.
3. **Choosing k** — The script fits K-means for k = 2 through 10 and plots:
   - **Elbow method**: inertia (within-cluster sum of squares) vs. k
   - **Silhouette score**: how well-separated the clusters are vs. k

   It automatically picks the k with the best silhouette score, though you
   can also read the elbow plot yourself and override it.
4. **Fitting** — Final `KMeans` model is fit on the scaled features with the
   chosen k.
5. **Naming clusters** — Each cluster is labeled automatically based on
   whether its median income/spending fall above or below the customer-wide
   median, e.g. *"High Income, High Spenders (Target)"*.
6. **Visualization** — Three plots: income vs. spending score (2D, most
   interpretable), a PCA projection using all features, and a bar chart
   comparing cluster averages.

## Files

| File | Description |
|---|---|
| `mall_customer_segmentation.py` | The full pipeline — run it end to end |
| `customer_segments.csv` | Every customer with its assigned `Cluster` and `SegmentName` |
| `elbow_plot.png` | Elbow + silhouette plots used to choose k |
| `cluster_plot_income_spend.png` | Income vs. Spending Score, colored by segment |
| `cluster_plot_pca.png` | All features (Age, Income, Spending) reduced to 2D via PCA |
| `cluster_profile.png` | Bar chart comparing average Age/Income/Spending per segment |

## How to run

```bash
pip install pandas numpy scikit-learn matplotlib seaborn
python mall_customer_segmentation.py Mall_Customers.csv
```

By default the script looks for `Mall_Customers.csv` in the same folder if
no path is given. It prints a summary table to the console and writes all
output files next to the script.

To force a specific number of clusters instead of the auto-selected one,
edit the call at the bottom of the script:

```python
main(csv_path="Mall_Customers.csv", k=5)
```

## Results (this run)

With this dataset, the silhouette score peaked at **k = 6**
(silhouette ≈ 0.43). Segment summary:

| Segment | Customers | Avg Age | Avg Income (k$) | Avg Spending Score |
|---|---|---|---|---|
| High Income, High Spenders (Target) | 39 | 32.7 | 86.5 | 82.1 |
| Low Income, High Spenders (Impulsive) | 23 | 25.0 | 25.3 | 77.6 |
| Low Income, Low Spenders (Budget) | 45 | 56.3 | 54.3 | 49.1 |
| Low Income, High Spenders (Impulsive, older) | 39 | 26.8 | 57.1 | 48.1 |
| Low Income, Low Spenders (Budget, older) | 21 | 45.5 | 26.3 | 19.4 |
| High Income, Low Spenders (Cautious) | 33 | 41.9 | 88.9 | 17.0 |

### How to read these segments

- **High Income, High Spenders (Target)** — the highest-value segment.
  Prioritize for premium products, early access, and loyalty perks — they
  already spend freely and have room to spend more.
- **High Income, Low Spenders (Cautious)** — high earning power but low
  engagement. Good target for personalized offers or re-engagement
  campaigns to convert latent spending power into purchases.
- **Low Income, High Spenders (Impulsive)** — spend a large share of a
  smaller income. Respond well to promotions, discounts, and
  limited-time deals.
- **Low Income, Low Spenders (Budget)** — price-sensitive; best reached
  with value bundles and clearance/discount messaging rather than premium
  campaigns.

Note: exact cluster counts and boundaries can shift slightly between runs
if you change `k` or the feature set (e.g., dropping `Age` collapses this
back to the classic 5-segment "income vs. spending" story).

## Notes & next steps

- K-means assumes roughly spherical, similarly-sized clusters in the
  scaled feature space — it works well here because Income and Spending
  Score form visually distinct groups, but validate with silhouette score
  if you swap in different features.
- Consider adding `Gender` as a comparison dimension (not a clustering
  feature) to see if segment composition differs by gender.
- If you have real transaction history instead of a single Spending Score,
  an RFM-based version of this pipeline (Recency/Frequency/Monetary) would
  give a more behavior-grounded segmentation — happy to build that version
  too if useful.
