# Frequently Asked Questions

## Model Fitting

**Q: Why isn't my random effects syntax working?**

A: Use the proper syntax from our testing:
```python
# This doesn't work
random = ~ 1|site

# Use this instead
re(random=~1|as.factor(site))
```

**Q: Do I need `+ as.factor(sex)` next to `pvc(age, by = as.factor(sex))`?**

A: No. With a factor `by=`, `pvc()` already fits each sex its own complete curve,
including its own level, so the extra main effect gives the same fit with one more
degree of freedom. It is harmless but not required:
```python
y ~ pvc(age, by = as.factor(sex))                    # enough
y ~ pvc(age, by = as.factor(sex)) + as.factor(sex)   # same fit, 1 extra df
```

**Q: What's the difference between `pb()` and `pvc()`, and which should I use?**

A: Use `pvc()` -- this package's examples use it throughout, even for plain
smooths with no `by=`. GAMLSS has two known quirks around factor-varying
smooths: `pb()` silently ignores `by=` and fits one shared curve (`Gamlss.fit()`
warns about this, but the fit is still a single curve, so use `pvc()` for
per-group curves), and `pvc(..., by=..., df=<fixed>)` used to fail when
predicting on new data (patched automatically by this package).

**Q: Can I write `log(y)` (or another transform of the outcome) in the formula?**

A: No, transform the column instead. NormGAMLSS reads the outcome column itself (`y`)
when it calculates z-scores, centiles and evaluation metrics, so with `log(y)` in the
formula it would compare the raw values with a distribution fitted to `log(y)`:
```python
# This gives wrong z-scores and centiles
log(y) ~ pvc(age) + re(random=~1|as.factor(site))

# Do this instead: a new column, fitted as log_y ~ pvc(age) + ..., with y_val='log_y'
df['log_y'] = np.log(df['y'])
```

**Q: Which distribution families are supported?**

A: The continuous and count families of `gamlss.dist` (e.g. NO, SHASH, BCT, GA, NBI).
Binomial families (BI, BB, ZIBI, ZABI, ZIBB, ZABB, DBI) are not supported, because
they need the number of trials (binomial denominator) for every observation, which
NormGAMLSS does not pass on. Multinomial families (MN3, MN4, MN5, MULTIN) have no
centiles or z-scores.

## Interpretation

**Q: How do I get actual parameter values from coefficients?**

A: Apply the inverse link function to the linear predictor, i.e. the sum of all terms for a
person, not to each coefficient (exp(intercept) is the value only when there are no other
terms). `model.predict_all()` does this for you:
```python
# For log link (sigma, nu, tau)
actual_value = exp(linear_predictor)

# For identity link (mu)
actual_value = linear_predictor
```

**Q: What do the diagnostic statistics mean?**

A: Target values for well-calibrated quantile residuals:
- Mean ≈ 0, Variance ≈ 1, Skewness ≈ 0, Kurtosis ≈ 3 (R's summary reports plain
  kurtosis; `evaluate()` reports *excess* kurtosis, whose target is ≈ 0)

See the [Diagnostic Plots Interpretation](diagnostic_plot_summary) page for details.

**Q: Which value does `evaluate_ml()` use as the prediction?**

A: The fitted `mu` of each person. For symmetric families such as NO, mu is the mean.
For skewed families such as SHASH, mu is the location, not the mean, so MAE, RMSE and
R² there describe the distance from the location. Use the z-score metrics of
`evaluate()` to judge the fit of the whole distribution.

**Q: How small can a permutation p-value be?**

A: `run_models_with_permutation()` calculates p = (1 + k) / (1 + n), where n is the
number of permutations that fitted and k the number with a difference at least as
large as the original one. p is never 0. The smallest possible value is 1 / (n + 1):
0.0196 with 50 permutations, 0.001 with 999. When you test many regions and correct
for multiple testing, use 1000 or more permutations.

## Technical Issues

**Q: Why am I getting plotting errors?**

A: Always wrap R plots:
```python
grdevices.png(file="plot.png", width=600, height=400)
base.plot(robjects.r['model_name'])
grdevices.dev_off()
```

**Q: Can I share my model without sharing the training data?**

A: Yes, with a slim copy. A model saved by `save_model()` (or by the pipelines) holds
every training participant's outcome and fitted values, and it needs the training data
to predict. `export_slim()` writes a small copy without any participant data that predicts
on its own (`train_data=None`); only diagnostics (residuals, QQ and worm plots) need the
full model. See *Sharing a model without the training data* in
[Saving and Loading Models](notebooks/saving_loading_models).
```python
m = Gamlss.load_model('saved_models/thick_formula3.rds')        # the full model
m.export_slim(train_data, 'share/thick_formula3_slim.rds')        # once, with its training data

slim = Gamlss.load_model('thick_formula3_slim.rds')              # anywhere, no data needed
z = slim.z_score(new_data, None)
```

**Q: How do I handle new sites?**

A: Use the site transfer function in `site_transfer_utils` (it is a module-level function, not a method of `Gamlss`):
```python
from normgamlss.site_transfer_utils import site_transfer

result = site_transfer(main_model_path, main_traindata, new_data, new_site,
                       selected_site, sample_size=20, y_val=y_val, x_vals=x_vals)
```
See the function's docstring for the sampling strategy, `params_to_adjust` and `adjustment_covariates` options.
With an adjustment covariate, `covariate_params` chooses which parameters get it (by
default, every adjusted parameter). With a small calibration sample (e.g. 20 people), try
adjusting the location only, `covariate_params=['mu']`: fewer terms to estimate, so less
risk of overfitting and of widened z-scores. With more people, the default (all adjusted
parameters) usually gives the bigger benefit.

## Model Selection

**Q: Which distribution should I use?**

A: Start with SHASH for flexibility, use `fitDist()` for data-driven selection.

**Q: How many iterations do I need?**

A: Check whether the fit converged (the model summary, or the 'Converged' columns of
the pipelines). RS alone can need hundreds of iterations, for example with SHASH, whose
shape parameters move slowly. `method = mixed(10, 100)` (10 RS iterations, then CG)
often converges in far fewer iterations; in the tutorial data RS(50) and RS(200) did
not converge, while mixed(10, 100) converged within 50 iterations. It does not always win:
on the Quick Start data RS converged in 56 iterations and mixed(10, 100) did not. If one
does not converge, try the other.

## Getting Help

- Check the [GAMLSS website](https://www.gamlss.com/) for R documentation
- Use `robjects.r('help(function_name)')` for R function help
- Review diagnostic plots carefully
- Start simple and add complexity gradually
