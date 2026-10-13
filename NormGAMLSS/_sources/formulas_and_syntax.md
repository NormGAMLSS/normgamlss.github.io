# GAMLSS Formula Reference and Syntax

Comprehensive guide to GAMLSS model specification, including formula syntax, random effects, and best practices.

## Basic Syntax

```python
y ~ predictors                    # For mu parameter
sigma.formula = ~ predictors      # For sigma parameter
nu.formula = ~ predictors         # For nu parameter  
tau.formula = ~ predictors        # For tau parameter
```

## Predictor Types

### Linear Terms
```python
y ~ age + sex                     # Linear effects
```

### Factor Variables
```python
y ~ as.factor(site)              # Categorical variable
y ~ as.factor(sex)               # Binary factor
```

### Smoothing Terms
```python
y ~ pvc(age)                     # P-spline smooth
y ~ pvc(age, df=5)               # Specify degrees of freedom
```

### Interactions
```python
y ~ pvc(age, by = as.factor(sex)) # Separate curves by sex
y ~ age * sex                    # Linear interaction
```

### Random Effects (Working Syntax)
```python
# Method 1
y ~ random(as.factor(site))

# Method 2 (Preferred)  
y ~ re(random=~1|as.factor(site))
```

## Complete Example

```python
model <- gamlss(
    y ~ pvc(age, by = as.factor(sex)) + re(random=~1|as.factor(site)),
    sigma.formula = ~ pvc(age) + re(random=~1|as.factor(site)),
    nu.formula = ~ 1,
    tau.formula = ~ 1,
    family = SHASH(),
    method = RS(50)
)
```

## Important Notes

- Use `as.factor()` for categorical numeric variables
- Start with simpler formulas and add complexity


## Random Effects: The Critical Issue

### ❌ This Doesn't Work (Despite Appearing To)
```r
model <- gamlss(y ~ pvc(age) + site,
    random = ~ 1|site,  # Shows in summary but doesn't actually work
    family = SHASH())
```

After extensive testing, the `random = ~ 1|site` parameter doesn't work as intended. Even though the model summary shows random effects are specified, the coefficients and degrees of freedom indicate it's not actually applying random effects.

### ✅ Working Alternatives

**Alternative #1: Using `random()` function**
```r
model <- gamlss(y ~ pvc(age, by = as.factor(sex)) + random(as.factor(site)),
    sigma.formula = ~ pvc(age) + random(as.factor(site)),
    family = SHASH())
```

**Alternative #2: Using `re()` function (Preferred)**
```r
model <- gamlss(y ~ pvc(age, by = as.factor(sex)) + re(random=~1|as.factor(site)),
    sigma.formula = ~ pvc(age) + re(random=~1|as.factor(site)),
    family = SHASH())
```

### Why `re()` is Preferred
- More effective in fitting the model
- Yields lower degrees of freedom
- Better model performance
- Allows use of full capabilities of the lme package
- Enables stronger priors for random effects

## Understanding Formula Components

### The `by` Parameter
`pvc(age, by = as.factor(sex))` creates **separate smoothing curves** for each sex level:
- If sex has 2 levels (0 and 1), the model estimates two different age curves
- The effect of age is allowed to differ completely between sex groups
- The smoothness of each curve is estimated separately

**Without interaction**: Assumes age affects outcome identically regardless of sex <br>
**With interaction**: Allows for potentially different age patterns for each sex category

**Note:** GAMLSS has two known quirks around factor-varying smooths:

- `pb(x, by=...)`: `pb()` has no `by` argument and silently ignores it, so it
  fits ONE shared curve instead of one per group. `Gamlss.fit()` detects this
  and raises a warning, but the fit itself is still a single curve -- use
  `pvc(x, by=...)` for group-specific curves.
- `pvc(x, by=..., df=<fixed>)` used to fail when predicting on new data. This
  package patches that automatically before prediction, so it now just works.

So `pvc(x, by=<factor>)` works as shown above.

### One-Hot Encoding in R

**For string variables**: R handles automatically
```r
# These are equivalent for string site variables:
y ~ pvc(age) + site
y ~ pvc(age) + as.factor(site)
```

**For numeric categorical variables**: Always use `as.factor()`
```r
# Wrong: treats scanner_id as continuous (assumes scanner 6 is "twice as far" from scanner 3)
y ~ pvc(age) + scanner_id

# Correct: treats scanner_id as categorical
y ~ pvc(age) + as.factor(scanner_id)
```

## Advanced Formula Patterns

### Do You Need a Main Effect Next to `pvc(x, by = <factor>)`?

```r
y ~ pvc(decimal_hour, by = as.factor(sex)) + as.factor(scanner_id)                   # enough
y ~ pvc(decimal_hour, by = as.factor(sex)) + as.factor(sex) + as.factor(scanner_id)  # same fit, 1 extra df
```

With a **factor** `by=`, `pvc()` fits each level its own complete curve, including
its own level (intercept), so adding `+ as.factor(sex)` does not change the fit. On
simulated data with a pure sex offset, both formulas gave identical predictions and
the same sex gap; the main effect only added one degree of freedom (AIC +2). Adding
it is harmless, but not required.

### Common Smoothing Terms
- **P-splines**: `pvc(x)` - Penalized B-splines (most common; also handles `by=` group-varying smooths, unlike the older `pb(x)`)
- **Cubic splines**: `cs(x)` - Cubic smoothing splines  
- **Neural networks**: `nn(x)` - Neural network smoothers

### Random Effects Patterns
```r
# Random intercept by site
re(random=~1|as.factor(site))

# Random slope and intercept
re(random=~age|as.factor(site))

# Multiple grouping factors
re(random=~1|as.factor(site)) + re(random=~1|as.factor(scanner))
```

## Distribution Families

### Common Choices
- **SHASH**: Flexible 4-parameter distribution (handles skewness + kurtosis)
- **NO**: Normal distribution (2 parameters: mu, sigma)
- **GA**: Gamma distribution (positive continuous data)
- **GG**: Generalized Gamma (very flexible)

### Family Examples
```r
family = SHASH()    # General purpose, very flexible
family = NO()       # When normality is reasonable
family = GG()       # Alternative flexible option
```

## Optimization Methods

### Available Methods
- **RS()**: Rigby and Stasinopoulos algorithm (robust, recommended)
- **CG()**: Cole and Green algorithm (faster, may need more iterations)
- **mixed()**: Combination approach

### Method Examples
```r
method = RS(50)        # 50 iterations, robust
method = CG(10)        # Faster convergence
method = mixed(5, 20)  # 5 RS iterations + 20 CG iterations
```

## Link Functions

Link functions connect the linear predictor to the parameter scale.

### Common Link Functions

From your model output, you'll see:

#### Identity Link
```
Mu link function: identity
```
- No transformation: parameter = linear predictor
- Used for location parameters that can be any real number

#### Log Link  
```
Sigma link function: log
```
- Ensures positive values: log(parameter) = linear predictor
- To get actual value: parameter = exp(estimate)
- Used for scale parameters (sigma, tau)

### Important Note

**Always apply inverse link to get actual parameter values:**

```python
# For log link
actual_sigma = exp(coefficient_estimate)

# For identity link  
actual_mu = coefficient_estimate
```

This is automatically handled in the `predict_all` function, but important to know for interpretation.
