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

## Interpretation

**Q: How do I get actual parameter values from coefficients?**

A: Apply the inverse link function:
```python
# For log link (sigma, nu, tau)
actual_value = exp(coefficient)

# For identity link (mu)
actual_value = coefficient
```

**Q: What do the diagnostic statistics mean?**

A: Target values for well-calibrated quantile residuals:
- Mean ≈ 0, Variance ≈ 1, Skewness ≈ 0, Kurtosis ≈ 3 (R's summary reports plain
  kurtosis; `evaluate()` reports *excess* kurtosis, whose target is ≈ 0)

See the [Diagnostic Plots Interpretation](diagnostic_plot_summary) page for details.

## Technical Issues

**Q: Why am I getting plotting errors?**

A: Always wrap R plots:
```python
grdevices.png(file="plot.png", width=600, height=400)
base.plot(robjects.r['model_name'])
grdevices.dev_off()
```

**Q: How do I handle new sites?**

A: Use the site transfer function in `site_transfer_utils` (it is a module-level function, not a method of `Gamlss`):
```python
from normgamlss.site_transfer_utils import site_transfer

result = site_transfer(main_model_path, main_traindata, new_data, new_site,
                       selected_site, sample_size=20, y_val=y_val, x_vals=x_vals)
```
See the function's docstring for the sampling strategy, `params_to_adjust` and `adjustment_covariates` options.

## Model Selection

**Q: Which distribution should I use?**

A: Start with SHASH for flexibility, use `fitDist()` for data-driven selection.

**Q: How many iterations do I need?**

A: Start with RS(50), increase to RS(100) if convergence issues.

## Getting Help

- Check the [GAMLSS website](https://www.gamlss.com/) for R documentation
- Use `robjects.r('help(function_name)')` for R function help
- Review diagnostic plots carefully
- Start simple and add complexity gradually
