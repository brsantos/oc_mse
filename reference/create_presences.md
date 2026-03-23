# Create matrix of presences and absences.

Based on a given population, \`create_presences()\` creates a matrix of
presences and absences based on a probability function to remain in the
country.

## Usage

``` r
create_presences(
  pop,
  varying_arrival = FALSE,
  const_rate_arrival = TRUE,
  formula_phi,
  coef_values,
  years = 2
)
```

## Arguments

- pop:

  A baseline population from which one will select individuals to arrive
  and leave in a number of years.

- varying_arrival:

  A logical parameter that controls if all individuals arrive in the
  first year or not. Default is FALSE.

- const_rate_arrival:

  In case individuals are arriving in different years, then if \`TRUE\`
  individuals will arrive at a constant rate each year. If \`FALSE\`
  then will arrive randomly in the different years.

- formula_phi:

  A linear predictor formula to calculate the probability of leaving the
  country.

- coef_values:

  The values for each coefficient to be used in the linear predictor
  defined in \`formula_phi\`. The coefficients will be used considering
  a logit link function.

- years:

  Number of years to define this matrix of absences and presences.

## Value

A matrix with the information of the presence (1) or absence (0) for
each individual of the population considering a number of years of
observation.

## Examples

``` r
main_pop <- create_population(
size = 500,
n_cat_var = 3,
prob_bin = c(0.5))

# example with all individuals arriving in the first year.
presences <- create_presences(main_pop,
  formula_phi = ~ bin1,
  coef_values = c(2, -1),
  years = 3)
colSums(presences)
#> [1] 500 399 329

# example with constant rate of arrival of individuals in each year.
presences <- create_presences(main_pop,
  formula_phi = ~ bin1,
  coef_values = c(2, -1),
  varying_arrival = TRUE,
  years = 3)
colSums(presences)
#> [1] 250 454 355

# example with varying rate of arrival of individuals in each year.
presences <- create_presences(main_pop,
  formula_phi = ~ bin1,
  coef_values = c(2, -1),
  varying_arrival = TRUE,
  const_rate_arrival = FALSE,
  years = 3)
colSums(presences)
#> [1] 224 456 369
```
