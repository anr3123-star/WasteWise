# WasteWise

**Forecasting meal demand to cut food waste in delivery kitchens.**

Won an Excellence Award at the Veritas AI International High School Hackathon 2026, top project out of 1,000+ submissions from 75+ countries. Built by a team of three in 48 hours.

## The finding

Most demand forecasting optimises accuracy. We asked a different question: what should a kitchen actually *do* with the forecast?

Converting predictions into kilograms of food and rupees of cost revealed that **accuracy, cost and food waste are three objectives that disagree**. No strategy wins on all three.

| Strategy | Test MAE | Kg wasted | Money lost (₹) |
|---|---|---|---|
| Same as last week | 79.13 | 540,161 | 563.7M |
| 4-week average | 81.70 | 600,283 | 556.6M |
| **Model, prep to forecast** | **64.52** | **388,647** | 490.6M |
| Model + 20% buffer | 75.08 | 749,378 | **393.3M** |

Preparing to the forecast is best on accuracy and waste: 18% lower error, 28% less food wasted, around 465 tonnes of CO2e avoided over ten weeks.

A 20% buffer costs least, because an unserved order loses the whole sale while a wasted meal loses only ingredients, roughly 3 to 1. But it nearly doubles waste.

We report both. The trade-off is real and the choice belongs to the kitchen.

## Method

Gradient boosting (`HistGradientBoostingRegressor`) on 456,548 rows of weekly demand. Features: price, discount size, promotion flags, kitchen and meal attributes, and each meal's recent sales at that kitchen.

Split strictly by time: train weeks 1–125, tune 126–135, test 136–145 touched once at the end. Every history feature is shifted and grouped per kitchen-meal pair, so no row sees the week it's predicting.

Waste conversion uses 0.4 kg per meal from USDA FNDDS portion weights, tested at 0.3/0.4/0.5 to confirm the ranking holds. Emissions use 3.07 kg CO2e per kg (FLW Protocol).

## Limitations

- The 3:1 cost ratio assumes a 30% food cost, which we could not verify
- The buffer is a post-hoc multiplier; training with a quantile loss would learn the cost structure directly and let it vary by meal
- One global buffer, though accuracy varies widely by category (rice bowls MAE 135, biryani 17)
- Portion weight is an assumption; no public dataset we found pairs measured kitchen waste with a serving count

## My contribution

Data pipeline and time-based validation split, the cost model converting forecast error into kilograms and rupees, the buffer analysis that produced the central finding, and the demo interface.

## Running it

Download `train.csv`, `fulfilment_center_info.csv` and `meal_info.csv` from [Genpact Food Demand Forecasting](https://www.kaggle.com/datasets/kannanaikkal/food-demand-forecasting) (free Kaggle account; ignore `test.csv`, it has no labels). Open the notebook in Colab, upload the CSVs, update the paths in the first cells.

## Sources

- [Genpact Food Demand Forecasting](https://www.kaggle.com/datasets/kannanaikkal/food-demand-forecasting), [University Canteen Food Waste](https://www.kaggle.com/datasets/rahuljangir78/food-waste-dataset-from-a-university-canteen) (Kaggle)
- [USDA FNDDS](https://www.ars.usda.gov/ARSUserFiles/80400530/pdf/fndds/2021_2023_FNDDS_Doc.pdf), [FLW Protocol](https://flwprotocol.org), [UNEP Food Waste Index](https://www.unep.org/en/what-we-do/food-systems/food-loss-and-waste)
- scikit-learn, pandas, NumPy, Gradio
- Claude, used for debugging and feedback on framing the waste metric
