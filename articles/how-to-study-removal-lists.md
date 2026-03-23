# How to study the effect of removing one list in the estimation of over-coverage

In the electronic supplementary material of our paper, there is a
simulation study where we discussed the different effects when removing
registers in the estimation of our model. The link and the info for the
paper follows:

- Mussino, E., Santos, B., Monti, A. et al. Multiple systems estimation
  for studying over-coverage and its heterogeneity in population
  registers. **Quality & Quantity** (2023).
  <https://doi.org/10.1007/s11135-023-01757-x>

Here we discuss the different steps of the simulation study and show how
to reproduce the experiment.

## Initial Setup

For this simultation, we need to generate new population according to
the same rules for every replication of the study. In the same way, we
create 3 lists, $X$, $Y$ and $Z$, where the probability of being
observed always follows the same function. For this experiment, we
create 3 binary variables that are used separately in the probability
function in each one of the lists.

The initial step is to generate the population, where we create one
continuous and one categorical variable, although they will not be used
for the study. The function just needs to create at least one variable
of these types.

``` r
library(ocmse)
set.seed(42)
main_list <- create_population(size = 1e6,
                                 n_bin_var = 3, 
                                 n_cont_var = 1,
                                 n_cat_var = 1,
                                 c(0.5))
```

We assume that individuals will leave the country according to a
logistic model and using `bin1` variable as the covariate to write this
probability. Using the `create_presences` function from our package and
considering only 2 years of observation, we have the following:

``` r
true_presences <- create_presences(main_list, 
                                     formula_phi = ~ bin1,
                                     coef_values = c(2, -1),
                                     varying_arrival = TRUE,
                                     years = 2)
```

If $B_{1}$ represents `bin1` and $\phi$ is the probability of staying in
the country (usually known as *surviving*, in the capture-recapture
literature) each year, we considering the following logistic regression
to calculate those probabilities

$$\log\begin{pmatrix}
\frac{\phi}{1 - \phi}
\end{pmatrix} = 2 - B_{1},$$

With this equation, we are creating two probabilities of staying in the
country, when $B_{1} = 0$ or when $B_{1} = 1$, which are calculated
respectively as

$$\frac{\exp(2)}{1 + \exp(2)} = 0.881\quad{\mspace{6mu}\text{and}\mspace{6mu}}\quad\frac{\exp(2 - 1)}{1 + \exp(2 - 2)} = 0.731.$$

For each simulation step we also create 3 lists, $X$, $Y$ and $Z$, using
the function `create_list_presences`.

``` r
X <- create_list_presences(main_list, presences = true_presences, 
                           formula_prob = ~ bin1, 
                           coef_values = c(1.5, -0.5))
   
Y <- create_list_presences(main_list, presences = true_presences,
                           formula_prob = ~ bin2, 
                           coef_values = c(-0.5, -0.5))
  
Z <- create_list_presences(main_list, presences = true_presences,
                           formula_prob = ~ bin3, 
                           coef_values = c(-1.5, -0.5))
```

The argument `formula_prob` controls which variables are used to
calculate the probability of being observed in each list. As we have
said previously, we are considering the 3 different binary variables
(`bin1`, `bin2` and `bin3`) separately for each list. The values in the
argument `coef_values` are used in the logistic regression to generate
the observation in each list. For instance, for list $X$ we are
calculating the following

$$\log\begin{pmatrix}
\frac{P\left( X = 1|B_{1} \right)}{1 - P\left( X = 1|B = 1 \right)}
\end{pmatrix} = 1.5 - 0.5 \cdot B_{1},$$

which gives us estimated probabilities of 0.818 and 0.731 for
$B_{1} = 0$ and $B_{1} = 1$, respectively.

Now we just need to recreate the scenarios where we use just part of
tesse lists to estimate the over-coverage. For this particular toy
example, the true value of over-coverage in the second year is given by

``` r
n_present <- sum(true_presences[, 2])
true_oc <- 1 - n_present/nrow(main_list)
true_oc
#> [1] 0.194634
```

## Removing lists and estimating over-coverage

At this step, we need to run the model to estimate the over-coverage and
repeat the same procedure removing one of the lists, for instance.
Considering all lists, we could run the following to estimate the
over-coverage

``` r
library(dplyr)
all_data <- 
  cbind(bin1 = main_list$bin1,
        bin2 = main_list$bin2,
        bin3 = main_list$bin3,
        X = X[, 2],
        Y = Y[, 2],
        Z = Z[, 2]) %>%
      as.data.frame()
    
freq_data <- all_data %>%
  group_by(bin1, bin2, bin3, X, Y, Z) %>%
  summarise(count = n())

freq_data <- freq_data %>%
  mutate(cens = ifelse(X == 0 &
                       Y == 0 &
                       Z == 0, TRUE, FALSE))

cens_ind <- which(freq_data$cens)
    
model <- ocmse::oc_model(
      model_formula = count ~ bin1 * X + bin2 * Y + bin3 * Z, 
      freq_table = freq_data, censored = cens_ind)
```

Given this model, we can estimate our over-coverage as

``` r
mean(model$oc_estimates)
#> [1] 0.1948825
```

And a credible interval with 95% of probability can be calculated as

``` r
quantile(model$oc_estimates, c(0.025, 0.975))
#>      2.5%     97.5% 
#> 0.1938549 0.1959254
```

A density plot of our estimates with the true value can be obtained with

``` r
library(ggplot2)

ggplot() + 
  theme_minimal() + 
  geom_density(aes(model$oc_estimates), adjust = 1.5, 
               fill = "royalblue") +
  geom_vline(aes(xintercept = true_oc, color = "True value"), linetype = 2) +
  labs(x = "Model-based over-coverage estimates") + 
  scale_color_manual(name = "", values = "red") + 
  theme(legend.position = "bottom")
```

![](how-to-study-removal-lists_files/figure-html/unnamed-chunk-10-1.png)

To estimate the model without list $X$ for instance, we would need to
run the following code. We need to change the censored observations
index before running the code

``` r
# removing X

freq_data_x <- all_data %>%
  group_by(bin1, bin2, bin3, Y, Z) %>%
  summarise(count = n())

freq_data_x <- freq_data_x %>%
  mutate(cens = ifelse(Y == 0 &
                         Z == 0, TRUE, FALSE))

cens_ind <- which(freq_data_x$cens)
    
model_x <- ocmse::oc_model(
      model_formula = 
        count ~ bin1 + bin2 * Y + bin3 * Z, 
      freq_table = freq_data_x, 
      censored = cens_ind, 
      nsample = 60000, 
      n.burnin = 50000, 
      thin = 10)
```

Now, our mean posterior estimates and 95% credible intervals will be
given by

``` r
mean(model_x$oc_estimates)
#> [1] 0.1962906
```

``` r
quantile(model_x$oc_estimates, c(0.025, 0.975))
#>      2.5%     97.5% 
#> 0.1905469 0.2019981
```

## Simulation setup

For the simulation, we will consider the same data set for every
replication of the study and we estimate over-coverage with all lists
and removing one list at a time. In order to make the simulations
easier, we write a function to run one replicate, where we create one
data set and return all the different over-coverage estimates.

``` r
sim_mse <- function(seed){
  
  set.seed(seed)

  n_years <- 2
  
  ## Creating population
  main_list <- create_population(size = 1e6,
                                 n_bin_var = 3, 
                                 n_cont_var = 1,
                                 n_cat_var = 1,
                                 c(0.5))
  
  true_presences <- create_presences(main_list, 
                                     formula_phi = ~ bin1,
                                     coef_values = c(2, -1),
                                     varying_arrival = TRUE,
                                     years = n_years)
  
  # calculating the true over-coverage at year 2.
  true_oc <- 1 - true_presences[, n_years] / nrow(main_list)
  
  X <- create_list_presences(main_list, 
                             presences = true_presences,
                             ~ bin1, 
                             c(1.5, -0.5))
   
  Y <- create_list_presences(main_list, 
                             presences = true_presences,
                             ~ bin2, 
                             c(-0.5, -0.5))
  
  Z <- create_list_presences(main_list, 
                             presences = true_presences,
                             ~ bin3, 
                             c(-1.5, -0.5))
  
  new_data <- 
  cbind(bin1 = main_list$bin1,
        bin2 = main_list$bin2,
        bin3 = main_list$bin3,
        X = X[, 2],
        Y = Y[, 2],
        Z = Z[, 2]) %>%
      as.data.frame()
  
  sim_cases <- list()
  
  freq_data <- new_data %>%
    group_by(bin1, bin2, bin3, X, Y, Z) %>%
    summarise(count = n())
  
  freq_data <- freq_data %>%
    mutate(cens = ifelse(X == 0 &
                         Y == 0 &
                         Z == 0, TRUE, FALSE))
  
  cens_ind <- which(freq_data$cens)
  
  sim_cases[[1]] <- oc_model(
    count ~ bin1 * X + bin2 * Y + bin3 * Z,
    freq_table = freq_data,
    cens = cens_ind, 
    nsample = 60000,
    n.burnin = 50000, 
    thin = 10)
  
  freq_data_x <- new_data %>%
    group_by(bin1, bin2, bin3, Y, Z) %>%
    summarise(count = n())
    
  freq_data_x <- freq_data_x %>%
      mutate(cens = ifelse(Y == 0 &
                           Z == 0, TRUE, FALSE))
    
  cens_ind_x <- which(freq_data_x$cens)
    
  sim_cases[[2]] <- oc_model(
    count ~ bin1 + bin2 * Y + bin3 * Z,
    freq_table = freq_data_x,
    cens = cens_ind_x, 
    nsample = 60000,
    n.burnin = 50000, 
    thin = 10) 
  
  freq_data_y <- new_data %>%
    group_by(bin1, bin2, bin3,
             X, Z) %>%
    summarise(count = n())
    
  freq_data_y <- freq_data_y %>%
    mutate(cens = ifelse(X == 0 &
                         Z == 0, TRUE, FALSE))
    
  cens_ind_y <- which(freq_data_y$cens)
    
  sim_cases[[3]] <- oc_model(
    count ~ bin1 * X + bin2 + bin3 * Z,
    freq_table = freq_data_y,
    cens = cens_ind_y, 
    nsample = 60000,
    n.burnin = 50000, 
    thin = 10) 

  freq_data_z <- new_data %>%
      group_by(bin1, bin2, bin3, X, Y) %>%
      summarise(count = n())
    
  freq_data_z <- freq_data_z %>%
    mutate(cens = ifelse(X == 0 & 
                           Y == 0, TRUE, FALSE))
    
  cens_ind_z <- which(freq_data_z$cens)
    
  sim_cases[[4]] <- oc_model(
    count ~ bin1 * X + bin2 * Y + bin3,
    freq_table = freq_data_z,
    cens = cens_ind_z, 
    nsample = 60000,
    n.burnin = 50000, 
    thin = 10)
  
  list(
    relat_mean = sapply(sim_cases, function(a) mean(a$oc_estimates)/true_oc),
    relat_q = sapply(sim_cases, function(a) 
      quantile(a$oc_estimates, c(0.025, 0.975))/true_oc))

}
```

Now we can write a function to call the same function a number of times
and record their results. We can do that with `lapply` or
[`purrr::map`](https://purrr.tidyverse.org/reference/map.html), but here
we will use the former, obtaining 50 replicates.

``` r
simulations <- lapply(1:50, sim_mse)
```

If we use a machine with multiple cores and a compatible operational
system, we can put this simulation study with multiple cores. For
instance, using the package `parallel`, we could run the following
defining the number of cores with the argument `mc.cores`.

``` r
simulations <- parallel::mclapply(1:50, sim_mse, mc.cores = 5)
```

## Collecting results and plotting

After running the simulations, we can gather the relative means and
credible intervals from each replicate and create a quick summary plot.
Below, we stack all results into data frames and make a boxplot of the
relative means (values around 1 indicate estimates close to the true
value).

``` r
scenarios <- c("All lists", "Remove X", "Remove Y", "Remove Z")

results_mean <- do.call(
  rbind,
  lapply(seq_along(simulations), function(i) {
    data.frame(
      sim = i,
      scenario = scenarios,
      rel_mean = simulations[[i]]$relat_mean
    )
  })
)

library(ggplot2)

ggplot(results_mean, aes(x = scenario, y = rel_mean)) +
  geom_boxplot(fill = "grey85") +
  geom_hline(yintercept = 1, linetype = 2, color = "red") +
  labs(x = "", y = "Relative mean (estimate / true)") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 15, hjust = 1))
```

If we also want to visualise the average 95% credible intervals across
replicates, we can summarize the lower and upper bounds and plot them as
error bars:

``` r
results_q <- do.call(
  rbind,
  lapply(seq_along(simulations), function(i) {
    data.frame(
      sim = i,
      scenario = scenarios,
      rel_low = simulations[[i]]$relat_q[1, ],
      rel_high = simulations[[i]]$relat_q[2, ]
    )
  })
)

summary_q <- results_q |>
  dplyr::group_by(scenario) |>
  dplyr::summarise(
    mean_low = mean(rel_low),
    mean_high = mean(rel_high),
    .groups = "drop"
  )

ggplot(summary_q, aes(x = scenario, y = 1)) +
  geom_hline(yintercept = 1, linetype = 2, color = "red") +
  geom_errorbar(aes(ymin = mean_low, ymax = mean_high), width = 0.2) +
  labs(x = "", y = "Average 95% interval (relative)") +
  theme_minimal() +
  theme(axis.text.x = element_text(angle = 15, hjust = 1))
```
