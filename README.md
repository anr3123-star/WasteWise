# WasteWise
Forecasting weekly meal demand to cut food waste in delivery kitchens.
Built as a team of 3 in 48 hours for the Veritas AI International High School Hackathon 2026 (AI + Climate track). Won an Excellence Award, top project out of 1,000+ submissions from 75+ countries.

## The problem
A meal delivery company decides every week how much of each meal to prepare, before any orders come in. Prepare too much and food goes in the bin. Prepare too little and customers go unserved.
Most kitchens just copy last week's numbers.

## What we found
Most demand forecasting projects optimise accuracy. We converted forecasts into kilograms of food and rupees of cost, and found that accuracy, cost and food waste are three different objectives that disagree.

| Strategy | Test MAE | Kg wasted | Money lost (₹) |
|---|---|---|---|
| Baseline: same as last week | 79.13 | 540,161 | 563.7M |
| Baseline: 4-week average | 81.70 | 600,283 | 556.6M |
| **Our model, prep to forecast** | **64.52** | **388,647** | 490.6M |
| Our model + 20% buffer | 75.08 | 749,378 | **393.3M** |

No single strategy wins on everything.

Preparing exactly to the forecast gives the best accuracy and the least waste: 18% lower error, 28% less food wasted, and roughly 465 tonnes of CO2e avoided over ten weeks.
Adding a 20% buffer costs the least money. An unserved order loses the whole sale while a wasted meal only loses ingredients, roughly a 3 to 1 ratio, so overpreparing is the cheaper mistake. But it nearly doubles food waste.
We report both and let the kitchen choose.

## Method

**Model:** gradient boosting (`HistGradientBoostingRegressor`) on 456,548 rows of weekly demand.
**Features:** price, discount size, promotion flags, kitchen and meal attributes, and each meal's recent sales at that kitchen.
**Split:** strictly by time. Train on weeks 1 to 125, tune on 126 to 135, test on 136 to 145 touched once at the end.
**Leakage control:** every history feature is shifted before use and grouped per kitchen-meal pair, so no row can see the week it's predicting.
**Waste conversion:** 0.4 kg per meal, from USDA FNDDS portion weights. Tested at 0.3, 0.4 and 0.5 to confirm the ranking of strategies holds either way.
**Emissions:** 3.07 kg CO2e per kg of food, the FLW Protocol weighted average.
## Running it
The notebook needs three CSVs from the Genpact Food Demand Forecasting dataset on Kaggle.

1. Download from [kaggle.com/datasets/kannanaikkal/food-demand-forecasting](https://www.kaggle.com/datasets/kannanaikkal/food-demand-forecasting). A free account is required.
2. You need `train.csv`, `fulfilment_center_info.csv` and `meal_info.csv`. Ignore `test.csv`, it has no labels.
3. Optionally download [University Canteen Food Waste](https://www.kaggle.com/datasets/rahuljangir78/food-waste-dataset-from-a-university-canteen) for the portion weight cross-check.
4. Open the notebook in Google Colab, upload the CSVs through the file sidebar, and update the file paths in the first cells. They currently point at a Google Drive folder.

Data files aren't included here. They belong to their original publishers.

## Limitations
The 3 to 1 cost ratio rests on an assumed 30% food cost, not measured data.
The buffer was applied after training as a multiplier, rather than training with an asymmetric loss. A quantile loss would let the model learn the cost structure directly and vary the buffer by meal.
We used one global buffer for all meals, though accuracy varies widely by category. Rice bowls have an MAE of 135 against 17 for biryani.
Portion weight is an assumption. We couldn't find a public dataset pairing measured kitchen waste with a serving count.

## Sources

- [Genpact Food Demand Forecasting](https://www.kaggle.com/datasets/kannanaikkal/food-demand-forecasting) (Kaggle)
- [University Canteen Food Waste](https://www.kaggle.com/datasets/rahuljangir78/food-waste-dataset-from-a-university-canteen) (Kaggle)
- [USDA FNDDS portion weights](https://www.ars.usda.gov/ARSUserFiles/80400530/pdf/fndds/2021_2023_FNDDS_Doc.pdf)
- [FLW Protocol, Connecting Food Loss and Waste to GHG Emissions](https://flwprotocol.org)
- [UNEP Food Waste Index](https://www.unep.org/en/what-we-do/food-systems/food-loss-and-waste)
- scikit-learn, pandas, NumPy, Gradio
- Claude, used for debugging and for feedback on framing the waste metric
