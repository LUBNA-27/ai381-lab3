Model selection
The question

The model predicts which customers are likely to cancel (churn), so the team can contact them with an offer. A missed churner (false negative) is a lost customer; a false alarm only costs one unneeded offer.

Candidates

All three trained on data/churn_v1.csv (data_sha256 76a6447e764b), batch lab3c, same train/test split.

Candidate	Why I tried it	AUC	F1	F1 per slice (lowest)	p95 latency (ms)	Size (KB)	MLflow run ID
logreg	Simple linear baseline from Lab 1; fast, tiny, easy to explain	0.798	0.495	0.250 (one-year, two-year)	5.2	3.5	cdbedf58
random_forest	Captures non-linear interactions between features	0.759	0.454	0.222 (one-year)	27.6	3160	2ccc376a
gradient_boosting	Usually the most accurate family on tabular data	0.748	0.417	0.222 (two-year)	6.0	142	99b6b66f

F1 per contract type:

Candidate	month-to-month	one-year	two-year
logreg	0.543	0.250	0.250
random_forest	0.494	0.222	0.364
gradient_boosting	0.436	0.444	0.222
The champion

logreg. It has the best overall F1 (0.495) and AUC (0.798), the best lowest-slice F1 (0.25), and the best F1 on month-to-month customers (0.543), the largest group and the one with most churners. It is also about 900x smaller than the random forest (3.5 KB vs 3160 KB) and about 5x faster. With only five features, the more complex models did not learn anything extra; they did worse.

What I accept losing: gradient_boosting is clearly better on one-year customers (F1 0.444 vs 0.25).

My promotion rules
Latency budget (p95): 50 ms, because a churn score is not a hard real-time call, and the measurement is noisy: the same logreg measured 36.2 ms in batch lab3b and 5.2 ms in batch lab3c on the same Codespace. A tight budget (the default 25 ms) would have blocked logreg because of noise and promoted a worse model.
Minimum gain to replace the champion (F1 points): 0.01, because a smaller difference can come from the random split alone, and every promotion means a new image and a new deployment.
What I would check next

The one-year and two-year slices are small and have few churners in the test set, so their F1 values (0.22 to 0.44) are unstable. Before trusting the champion for those customers I would use cross-validation or more data, and watch F1 per contract type in production.