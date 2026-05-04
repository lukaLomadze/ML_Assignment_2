# ML_Assignment_2
## კონკურსის მოკლე მიმოხილვა

[IEEE-CIS Fraud Detection] —  კლასიფიკაციის კონკურსია, რომლის მიზანია  ტრანზაქციებიდან თაღლითური ოპერაციების ამოცნობა. შეფასების კრიტერიუმია ROC AUC.

მონაცემები ორ ფაილადაა: transaction და identity, რომლებიც TransactionID-თ არის დაკავშირებული. კლასები დაუბალანსებელია.


## ჩემი მიდგომა

cleaning — გადავაგდე ის სვეტები სადაც transaction-ში 80%-ზე მეტი, identity-ში კი 70%-ზე მეტი მნისვნელობა არ გვქონდა. 

feature engineering — TransactionAmt-ზე log/sqrt ტრანსფორმაციები, TransactionDT-დან საათი, კვირის დღე, კვირა, თვე. XGBoost_better-ში დავამატე შედარებით განსხვავებული სვეტები.

feature selection — კატეგორიული სვეტები cardinality-ის მიხედვით ვყოფდი. კორელაციის ფილტრი LR-სა და XGB-ისთვის. დაუბალანსებლობის გამო SMOTE და  RandomUnderSampler გამოვიყენე.

training — 5 არქიტექტურა გამოვიყენე: Logistic Regression, Decision Tree, Random Forest, AdaBoost(ამას ძალიან დიდი დრო დაწირდა და ბოლომდე არც გასულა), XGBoost. ყველა 3-Fold Stratified CV-ით.()


საბოლოო შედეგი Kaggle-ზე: Public  **0.928**, Private  **0.894**



## რეპოზიტორიის სტრუქტურა

```
ML_Assignment_2/
├── model_experiment_LR.ipynb
├── model_experiment_DT.ipynb
├── model_experiment_RF.ipynb
├── model_experiment_ADA.ipynb
├── model_experiment_XGB.ipynb
├── model_experiment_XGB_better.ipynb (model_experiment ფაილებში თითოეული მოდელისთვის შესაბამისი cleanint,feature engineering/selection, training და mlflow )
├── model_inference.ipynb (საუკეთესო მოდელის პაიპლაინის ჩამოტვირთვა და ტესტზე პროგნოზი)
└── README.md
```

## Cleaning



სვეტები, სადაც missing rate transaction-ში 80%-ს, identity-ში კი 70%-ს აღემატებოდა, მთლიანად ამოვიღე (DropHighMissing transformer). თავიდან დამერჯილიზე ვაპირებდი ,მარა რადგან identity 24% იყო სულ დიდი თრეშჰოლდი უნდა დამედო , რრომ identity  ს სვეტებიდან რამე დარჩენილიყო.

დარჩენილი NaN-ები — SimpleImputer(strategy="median").

TransactionID და isFraud სვეტები Feature Engineering ეტაპში ამოვიღე .




## Feature Engineering

**TransactionAmt**
- `TransactionAmt_log` 
- `TransactionAmt_sqrt`
- `TransactionAmt_round`
- `TransactionAmt_decimal`

**TransactionDT**
- `TransactionHour` 
- `TransactionDay` 
- `TransactionWeek`
- `TransactionMonth` 

**Identity**
- `has_identity` — აქვს თუ არა ტრანზაქციას identity ჩანაწერი 

**XGBoost_better-ში დამატებული:**
- `uid_tx_count`, `uid_amt_mean`, `uid_amt_std` — card1+addr1 კომბინაციაზე დაყრდნობით.
- `{col}_freq` — card1/card2/card3/card5/addr1/addr2-ის frequency encoding 

**კატეგორიული ცვლადების კოდირება:**

- დაბალი cardinality (≤ CARDINALITY_TH) — OneHotEncoder
- მაღალი cardinality (> CARDINALITY_TH) — OrdinalEncoder

CARDINALITY_TH განსხვავებულია სხვადასხვა მოდელებისთვის.



## Feature Selection

**CorrelationFilter**

CorrelationFilter  — გამოვიყენე იმ მოდელებისთვის რომლებშიც აზრი ექნებოდ.

**Imbalance handling**

SMOTE და RandomUnderSampler. ზოგადად SMOTEს  undersample-თან შედარებით CV AUC-ზე უკეთესი შედეგი ქონდა. XGBoost_better-ში scale_pos_weight გამოვიყენე და უკეთესი შედეგი მომცა.




## Training

ყველა მოდელისთვის — Manual Grid Search, 3-Fold Stratified Cross-Validation და  roc_auc შესაფასებლად რადგან kaggle ც იგივეს იყენებს.

თითოეული მოდელისთვის განვიხილე სხვადასხვა ჰიპერპარამეტრები სხვადასხვა კომბინაციებით. და თითოეული ტიპის საუკეთესო შედეგის მქონე მოდელი დავარეგისტრირე.

**საბოლოო მოდელის შერჩევა**

საბოლოოდ ავარჩიე xgboost მოდელი, რომლის cv_auc=0.94 . საბოლოო ტესტ დათაზე კი ასეთი შედეგი მივიღე : Public  **0.928**, Private  **0.894**


## MLflow Tracking

MLflow ზე დალოგილია თითოეული მოდელი ცალკე ექსპერიმენტად.
https://dagshub.com/lukaLomadze/ML_Assignment_2.mlflow/#/