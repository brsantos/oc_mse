# Create a simulated population to test models for overcoverage.

\`create_population()\` creates a simulated population with a defined
number number of binary, continuous and categorical variables.

## Usage

``` r
create_population(
  size,
  n_bin_var = 1,
  n_cont_var = 1,
  n_cat_var = 1,
  prob_bin,
  prob_cat = c(0.5, 0.3, 0.2),
  names_bin_var = NULL,
  names_cont = NULL,
  names_cat_var = NULL
)
```

## Arguments

- size:

  Size of population to be created.

- n_bin_var:

  Number of binary variables to be created in the population.

- n_cont_var:

  Number of continuous variables to be created in the population.
  Continuous variables are generated according to a Normal distribution.

- n_cat_var:

  Number of categorical variables to be created in the population.
  Categorical variables are always created with three categories: A, B
  and C.

- prob_bin:

  Vector with probabilities of each binary variable to be created. In
  case this does not have the same length of \`n_bin_var\` then the
  first value of the vector is repeated \`n_bin_var\` times.

- prob_cat:

  Vector with probabilities of categorical variables to be created. This
  must have length 3 and by default the values are 0.5, 0.3 and 0.2.

- names_bin_var:

  Vector of names for binary variables.

- names_cont:

  Vector of names for continuous variables.

- names_cat_var:

  Vector of names for categorical variables.

## Value

A \`data.frame\` with all the population with their respective
variables.

## Examples

``` r
# basic usage of create_population function
main_pop <- create_population(
  size = 1e6,
  n_cont_var = 1,
  n_cat_var = 3,
  prob_bin = c(0.5))
```
