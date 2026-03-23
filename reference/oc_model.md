# Model to estimate over-coverage on population

The \`oc_model()\` function is able to get an estimate of over-coverage
in a population based on register data and log-linear models.

## Usage

``` r
oc_model(
  model_formula,
  freq_table,
  censored,
  nsample = 2000,
  null.move.prob = 1,
  n.burnin = 1000,
  thin = 1,
  prob_level = 0.95,
  ...
)
```

## Arguments

- model_formula:

  Model formula to be used in the log-linear model.

- freq_table:

  Frequency table with all observational data.

- censored:

  Indexes of all individuals who are not observed in any of the
  registers.

- nsample:

  Number of posterior draws in the MCMC estimation process.

- null.move.prob:

  Parameter to control model selection algorithm. See
  \`conting::bict()\` for more information.

- n.burnin:

  Number of burnin samples to be discarded from the MCMC algorithm.

- thin:

  Thinning parameter in the MCMC algorithm.

- prob_level:

  Probability level to be used when calculating the credible interval
  for the overcoverage level.

- ...:

  Additional arguments to be passed to \`conting::bict()\` function,
  which is slightly changed here to function properly in the latest
  versions of R.

## Value

A list with overcoverage estimates and summaries of the number of false
positives based on estimates of the model. The object model itself is
also returned.

## Examples

``` r
if (FALSE) { # \dontrun{
model_oc <- oc_model(
 qty ~ bin1 * list1 + cat1 * list2,
 freq_table,
 cens_ind)} # }
```
