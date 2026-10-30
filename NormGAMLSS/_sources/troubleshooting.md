# Troubleshooting and Common Issues

Solutions to common problems encountered when working with GAMLSS models.

## Random Effects Issues

### Problem: Random effects not working as expected
**Symptoms**: Model summary shows random effects specified but coefficients and degrees of freedom suggest otherwise.

**❌ Problematic syntax:**
```r
random = ~ 1|site  # Doesn't work as intended
```

**✅ Working solutions:**
```r
# Option 1: Use re() function (preferred)
re(random=~1|as.factor(site))

# Option 2: Use random() function  
random(as.factor(site))
```

**How to verify random effects are working:**
1. Compare the model with and without the site term: a working term lowers the global
   deviance and usually AIC/BIC. With `random = ~ 1|site` the fit is identical to the model
   without site, even though `random` appears in the call
2. Do not rely on the degrees of freedom alone: with few sites and many people per site,
   `re()` uses almost as many as a fixed site effect (in a toy test with 6 sites of 150
   people: 11.5 vs 11.9)

### Problem: Site variables not properly encoded
**Issue**: Numeric site IDs treated as continuous rather than categorical.

**Solution**: Always use `as.factor()` for categorical variables:
```r
# Wrong: treats scanner_id as continuous
y ~ pvc(age) + scanner_id

# Correct: treats scanner_id as categorical  
y ~ pvc(age) + as.factor(scanner_id)
```

## Convergence Problems

### Model fails to converge
**Try these approaches in order:**

1. **Increase iterations or change the method**: neither always wins, so check `converged`
   after each:
   ```r
   method = RS(300)         # more RS iterations
   method = mixed(10, 100)  # 10 RS iterations, then up to 100 CG iterations
   ```
   CG on its own is not recommended: in a toy SHASH test `CG(50)` diverged.

2. **Simplify model temporarily**:
   ```r
   # Start with simpler model
   y ~ pvc(age) + as.factor(sex)
   # Add complexity gradually
   ```

3. **Check data quality**:
   ```python
   # Look for outliers
   df.describe()
   # Check for missing values
   df.isnull().sum()
   # Check for infinite values
   np.isinf(df.select_dtypes(include=[np.number])).sum()
   ```

4. **Scale predictors** (if R reports numerical errors; in a toy test, rescaling an outcome
   around 9,500 and a predictor around 60,000 did not change convergence):
   ```python
   # Standardize continuous variables
   from sklearn.preprocessing import StandardScaler
   scaler = StandardScaler()
   df['age_scaled'] = scaler.fit_transform(df[['age']])
   ```

### Convergence warnings
**Common causes and solutions:**

- **Insufficient iterations**: Increase iteration count
- **Poorly scaled predictors**: Standardize continuous variables
- **Overly complex model**: Reduce number of parameters relative to sample size
- **Numerical instability**: Check for perfect collinearity between predictors

## Data Issues

### Special characters in column names
**Problem**: R errors with column names containing "-", "&", or spaces.

**Solution**: Use the provided utility function:
```python
[modified_df], [name_map] = remove_special_characters([df])
# name_map shows original -> modified name mapping
```

### Site mismatch between train and test
**Problem**: Test data contains sites not in training data.

**Diagnosis**:
```python
check_test_sites(train_data, test_data)
```

**Solutions**:
1. **Add extra site data to training**:
   ```python
   train_data_updated, test_data_updated = add_extra_site_data(
       train_data, test_data, fraction=0.2
   )
   ```

2. **Use site transfer methodology** (see multisite transfer chapter)

### Missing or infinite values
**Check and handle problematic values:**
```python
# Comprehensive data check
print("Missing values:", df.isnull().sum().sum())
print("Infinite values:", np.isinf(df.select_dtypes(include=[np.number])).sum().sum())

# Remove problematic rows
df_clean = df.dropna()
df_clean = df_clean[~np.isinf(df_clean.select_dtypes(include=[np.number])).any(axis=1)]
```

## Model Specification Issues

### Too many parameters for sample size
**Symptoms**: 
- Very low residual degrees of freedom
- Overfitting warnings
- Poor generalization to test data

**Solutions**:
1. **Reduce smoothing complexity**:
   ```r
   pvc(age, df=3)  # Limit degrees of freedom
   ```

2. **Remove non-significant terms**:
   - Check p-values in model summary
   - Use stepwise selection with `stepGAIC()`

3. **Use random effects instead of fixed effects**:
   ```r
   # Instead of: as.factor(site) (one parameter per site)
   # Use: re(random=~1|as.factor(site)) (fewer effective parameters when sites are small;
   # with large sites the saving is small)
   ```

### Non-significant predictors
**When p-values > 0.05:**

**Consider removing**, but weigh against:
- Theoretical importance of the variable
- Impact on other model terms
- Overall model performance (AIC/BIC)

**Use model comparison**:
```python
# Compare models with and without the predictor
comparison_df, summary_df = model_comparison(...)
```

## Prediction Issues

### NaN values in predictions for new sites
**Cause**: Random effects model can't predict for unseen factor levels.

**Solutions**:
1. **Use site transfer methodology** (recommended)
2. **Add small amount of new site data to training**
3. **Use level=0 in re()** (but loses random effects for known sites)

### Extreme prediction values
**Diagnostic checks**:
1. **Input data ranges**: Ensure test data is within training data range
2. **No extrapolation**: Check age, volume ranges match training
3. **Link functions**: Verify transformations are applied correctly

**Example check**:
```python
# Compare ranges
print("Training age range:", train_data['age'].min(), "-", train_data['age'].max())
print("Test age range:", test_data['age'].min(), "-", test_data['age'].max())
```

## Performance Issues

### Slow model fitting
**Optimization strategies**:

1. **Start with fewer iterations for testing**:
   ```r
   method = RS(10)  # Quick test
   ```

2. **Use multiprocessing for batch operations**:
   ```python
   use_multiprocessing = True
   num_cores = 8
   ```

3. **Enable caching**:
   ```python
   force_overwrite = False  # Use cached results when available
   ```

4. **Simplify models during development**:
   ```r
   # Remove complex smoothing terms temporarily
   # Add back once basic model works
   ```

## R Integration Issues

### rpy2 connection problems
**Common fixes**:

1. **Check R installation**:
   ```python
   import rpy2.robjects as robjects
   print(robjects.r('R.version.string')[0])
   ```

2. **Verify package installation**:
   ```python
   # Check if gamlss is installed
   robjects.r('installed.packages()["gamlss",]')
   ```

3. **Restart Python session**: Sometimes clears connection issues

4. **Set environment variables** (if needed), before importing `rpy2` or `normgamlss`:
   ```python
   import os
   os.environ["R_HOME"] = "/path/to/R"  # if needed
   import rpy2.robjects as robjects     # only now
   ```

### Graphics issues
**For R plots in Python**:

```python
# Ensure proper device management
grdevices = importr('grDevices')

# Always close devices after plotting
grdevices.dev_off()

# Turn off interactive plotting if needed
plt.ioff()
```

