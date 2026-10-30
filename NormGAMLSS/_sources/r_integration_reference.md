# R Integration Reference

Complete reference for working with R functions and packages through rpy2 in the NormGAMLSS package.

## Important R Functions by Package

### From `gamlss` package:
- `gamlss()`: Main fitting function for GAMLSS models
- `stepGAIC()`: Stepwise model selection using Generalized AIC
- `centiles()`: Compute centiles/quantiles for fitted models
- `plot()`: Diagnostic plots for GAMLSS models
- `getSmo()`: Extract smoothing parameters
- `predictAll()`: Generate predictions for new data
- `fitDist()`: Fit parametric distributions to data
- `histDist()`: Histogram with fitted distribution overlay
- `pdf.plot()`: Plot probability density functions

### From `stats` package:
- `AIC()`: Akaike Information Criterion
- `BIC()`: Bayesian Information Criterion  
- `coef()`: Extract model coefficients
- `fitted()`: Extract fitted values
- `residuals()`: Extract model residuals
- `deviance()`: Compute model deviance

### From `base` package:
- `summary()`: Model summary statistics
- `plot()`: Basic diagnostic plots

## Accessing Model Parameters

**Get all available model components** (`model_name` is the name you gave `Gamlss(model_name, ...)`):
```python
keys = robjects.r.names(robjects.r[model_name])
print(keys)
```

**Key model parameters include:**
- `'aic'`: Akaike Information Criterion
- `'sbc'`: Schwarz Bayesian Criterion (BIC)
- `'G.deviance'`: Global deviance
- `'mu.coefficients'`: Coefficients for mu parameter
- `'sigma.coefficients'`: Coefficients for sigma parameter  
- `'mu.df'`: Effective degrees of freedom for mu
- `'converged'`: Convergence status
- `'call'`: Model call/formula used

**Extract specific parameters:**
```python
# Information criteria
aic = robjects.r[model_name].rx2('aic')[0]
bic = robjects.r[model_name].rx2('sbc')[0]

# Coefficients
mu_coef = robjects.r[model_name].rx2('mu.coefficients')
sigma_coef = robjects.r[model_name].rx2('sigma.coefficients')

# Model fit statistics
global_deviance = robjects.r[model_name].rx2('G.deviance')[0]
degrees_freedom = robjects.r[model_name].rx2('df.fit')[0]

# Convergence check
converged = robjects.r[model_name].rx2('converged')[0]
```

## Advanced R Integration Techniques

### Checking Random Effects Implementation

Seeing `random` in the model call is no proof that a random effect was fitted: a
`random = ~1|site` argument to `gamlss()` appears in the call but is ignored. Compare the
model with and without the site term instead (`model_no_site` is the same model fitted
without it); a working site term lowers the global deviance and usually AIC/BIC:
```python
for name in [model_name, 'model_no_site']:
    m = robjects.r[name]
    print(name, 'deviance', m.rx2('G.deviance')[0], 'df', m.rx2('df.fit')[0],
          'AIC', m.rx2('aic')[0], 'BIC', m.rx2('sbc')[0])
```
For a `re()` term, the value stored in `mu.lambda` is not a shrinkage parameter, so do not use
it for this check.

### Using R Graphics in Python Notebooks
```python
from rpy2.robjects.packages import importr
from IPython.display import Image
import rpy2.robjects as robjects

# Import graphics packages
grdevices = importr('grDevices')
graphics = importr('graphics')

# Create plot
grdevices.png(file="diagnostic_plot.png", width=800, height=600, res=150)

# R plotting commands
robjects.r(f'''
# Your R plotting code here
plot({model_name})
''')

# Close device and display
grdevices.dev_off()
Image(filename='diagnostic_plot.png')
```

### Error Handling with R Functions
```python
try:
    # Attempt R operation
    result = stats.BIC(robjects.r[model_name])[0]
except Exception as e:
    print(f"Error calling R function: {e}")
    # Fallback or alternative approach
```

### Converting R Objects to Python
```python
from rpy2.robjects import pandas2ri
from rpy2.robjects.conversion import localconverter

# Scope the conversion to just the block that needs it, rather than
# activating it globally -- this is the pattern used throughout this
# package (see Gamlss.fit(), .predict_all(), etc.)
with localconverter(robjects.default_converter + pandas2ri.converter):
    r_dataframe = robjects.conversion.py2rpy(my_pandas_dataframe)
    # ... work with r_dataframe here ...
    python_dataframe = robjects.conversion.rpy2py(some_r_dataframe)
```

## Debugging R Integration Issues

### Check R Environment
```python
# Verify R packages are installed
robjects.r('installed.packages()["gamlss",]')

# Check R version
print(robjects.r('R.version.string')[0])

# List loaded packages
print(robjects.r('search()'))
```

### Common Integration Problems

**Issue**: R functions not found
**Solution**: Ensure packages are properly imported
```python
# Explicit package import
gamlss = importr('gamlss')
stats = importr('stats') 
base = importr('base')
```

**Issue**: Graphics not displaying
**Solution**: Use proper device management
```python
# Always close graphics devices
grdevices.dev_off()
# Use non-interactive plotting
plt.ioff()
```

**Issue**: Memory issues with large models
**Solution**: Explicit cleanup
```python
# Remove R objects you no longer need (rm(list=ls()) would also delete
# every fitted/loaded model that your Gamlss objects point to)
robjects.r('rm(my_large_object)')
# Force garbage collection
robjects.r('gc()')
```
