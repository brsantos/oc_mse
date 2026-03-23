# Creation of list of people that deregister from the population.

Given a population and a list of presences, \`create_list_out()\`
creates a list of individuals who deregister.

## Usage

``` r
create_list_out(pop, presences, departures, formula_prob, coef_values)
```

## Arguments

- pop:

  A baseline population from which one will select individuals to arrive
  and leave in a number of years.

- presences:

  A matrix of presences of all the individuals in the population for a
  number of years.

- departures:

  A vector of departure times of all the individuals in the population
  for a number of years.

- formula_prob:

  A linear predictor formula to calculate the probability of
  deregistering.

- coef_values:

  The values for each coefficient to be used in the linear predictor
  defined in \`formula_prob\`. The coefficients will be used considering
  a logit link function.

## Value

A matrix with the information of the deregistering (1) or non non
deregistering (0) for each individual of the population considering a
number of years of observation.
