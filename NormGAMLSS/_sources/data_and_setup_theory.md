# Understanding R Integration and External Packages

This chapter covers the R integration aspects and available packages that complement the practical notebooks.

## Model Name Configuration

> **💡 Best Practice:** 
> Set your model name as a variable at the beginning to use consistently throughout your analysis:
> ```python
> # Set this to match the model_name you used when creating your Gamlss instance
> model_name = 'my_model'  # Change this to match your actual model name
> 
> # Example: If you created your model like this:
> # model = Gamlss(model_name='my_custom_model', ...)
> # Then set: model_name = 'my_custom_model'
> ```

## External R Packages Available

Beyond the core `gamlss`, `base`, and `stats` packages, several additional R packages extend GAMLSS functionality:

**Core GAMLSS Packages:**
- `gamlss.dist`: 100+ distribution families (gamma, Weibull, beta, etc.)
- `gamlss.add`: Extra smooth functions: tensor products, neural networks and decision trees
  (penalized splines such as `pb()` and `pvc()` are part of `gamlss` itself)
- `gamlss.tr`: Truncated versions of distributions
- `gamlss.cens`: Tools for censored response variables
- `gamlss.data`: Example datasets

**Import packages in Python:**
```python
gamlss_dist = importr('gamlss.dist')
gamlss_add = importr('gamlss.add')
# Install if needed:
# robjects.r('install.packages(c("gamlss.dist", "gamlss.add"))')
```

**Explore package functions:**
```python
functions_list = robjects.r('ls(pos = "package:stats")')
print(functions_list)

# Get help for specific functions:
stats = importr('stats')
function_help = robjects.r('help(AIC)')
```

## Two Ways of Using R Functions

**Method 1: Through imported packages (Recommended)**
```python
# Import the package first
stats = importr('stats')
base = importr('base')

# Use the function with your model
bic = stats.BIC(robjects.r[model_name])[0]
model_summary = base.summary(robjects.r[model_name])
```

**Method 2: Direct R calls**
```python
# Direct call (sometimes has display issues)
robjects.r['BIC'](robjects.r[model_name])

# For Python output, convert the result explicitly. Avoid pandas2ri.activate() --
# it's removed in rpy2 3.6+ (see the Installation Guide); use localconverter instead,
# the same pattern used throughout this package.
from rpy2.robjects import pandas2ri
from rpy2.robjects.conversion import localconverter

bic_r = robjects.r['BIC'](robjects.r[model_name])   # call R outside the converter block
with localconverter(robjects.default_converter + pandas2ri.converter):
    result = robjects.conversion.rpy2py(bic_r)
```

## Plotting R Graphics in Python

```python
from rpy2.robjects.packages import importr
from IPython.display import Image

grdevices = importr('grDevices')

# Initialize graphics device
grdevices.png(file="plot.png", width=600, height=400)

# Your plotting code here
base.plot(robjects.r[model_name])

# Close device and display
grdevices.dev_off()
Image(filename='plot.png')
```

## Getting Model Parameters

**Set your model name first:**
```python
# Change this to match your actual model name from Gamlss(model_name='...')
model_name = 'my_model'
```

**View all available parameters:**
```python
keys = robjects.r.names(robjects.r[model_name])
print(keys)
```

**Access specific parameters:**
```python
aic_value = robjects.r[model_name].rx2('aic')[0]
coefficients = robjects.r[model_name].rx2('mu.coefficients')
global_deviance = robjects.r[model_name].rx2('G.deviance')[0]
```