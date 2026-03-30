## inf.ranger: Modified ranger for inference forests
`inf.ranger` is a minimal fork of [ranger](https://github.com/imbs-hl/ranger) (Wright & Ziegler, 2017) with targeted modifications to support nonparametric inference from random forests. All other ranger internals — tree growing, prediction, survival forests, variable importance, and the C++/R interface — are unchanged.

### Modifications
**Honest estimation** (`honest = TRUE`). Standard random forest trees compute leaf predictions from the same observations used to choose the splits, creating a dependence between the partition structure and the estimation sample that invalidates naïve inference. Honest forests break this dependence at the tree level: each tree subsamples 50% of the training data for split selection and recomputes leaf values from the held-out 50%. Setting `honest = TRUE` automatically enforces `sample.fraction = 0.5` and `replace = FALSE`; no other parameter changes are needed. Leaf mean estiamtes are then based on the held-out fraction as opposed to that used for training/building the tree. The result is a single forest whose OOB predictions are fully honest — every observation's prediction comes from trees where it was excluded from both structure and estimation. This is the foundation for valid variance estimation, confidence intervals, and effect inference in the companion `infForest` package.

**Standardized splitting criterion** (`penalize.split.competition = TRUE`). At each node, CART selects the variable with the largest impurity reduction. But continuous variables evaluate many more candidate split points than binary or low-cardinality variables, giving them a structural search advantage. The standardized criterion subtracts the expected impurity gain under the null (no signal) from each variable's best split score, producing a level comparison across variable types. The correction is closed-form and adds negligible computation.

**Softmax variable selection** (`softmax.split = TRUE`). Standard CART selects the single best variable at each node (argmax). Softmax replaces this with probabilistic selection: variables are chosen with probability proportional to their standardized criterion scores. This increases the inclusion rate for variables with moderate but real signal. Requires `penalize.split.competition = TRUE`.

The standardized criterion and softmax selection operate only at the moment of variable selection within each node. Honest estimation operates as a post-processing step after tree growing. Everything else — split point optimization, daughter node assignment, and all prediction methods — is identical to standard ranger.
### Installation
```r
devtools::install_github("NateOConnellPhD/inf.ranger")
```
### Usage
```r
library(inf.ranger)

# Standard ranger call — fully compatible
ranger(y ~ ., data = dat, num.trees = 5000)

# Honest forest — unbiased leaf estimates for inference
ranger(y ~ ., data = dat, num.trees = 5000, honest = TRUE)

# With standardized splitting
ranger(y ~ ., data = dat, num.trees = 5000, penalize.split.competition = TRUE)

# With softmax selection
ranger(y ~ ., data = dat, num.trees = 5000,
       penalize.split.competition = TRUE, softmax.split = TRUE)

# Honest + standardized + softmax (full inference configuration)
ranger(y ~ ., data = dat, num.trees = 5000,
       honest = TRUE, penalize.split.competition = TRUE, softmax.split = TRUE)
```
### Related packages
- **[infForest](https://github.com/NateOConnellPhD/infForest)** — Nonparametric inference from random forests using `inf.ranger` as its forest engine. Effect estimation, prediction intervals, variance decomposition, and uncertainty quantification.
- **[ranger](https://github.com/imbs-hl/ranger)** — The original fast random forest implementation by Wright & Ziegler. See their documentation for full details on all ranger features and parameters.
### References
- Wright, M. N. & Ziegler, A. (2017). ranger: A fast implementation of random forests for high dimensional data in C++ and R. *J Stat Softw* 77:1-17. https://doi.org/10.18637/jss.v077.i01.
- O'Connell, N. S. (2026). Random Forests as Statistical Procedures: Design, Variance, and Dependence. *arXiv:2602.13104*. https://arxiv.org/abs/2602.13104