# Convergence Problems

GAMLSS models may fail to converge when the optimization algorithm cannot find a stable solution. This systematic guide helps you diagnose and fix convergence issues step by step.

## Step 1: Check if Your Model Actually Converged

**Always start here** - some might assume convergence failed when it actually succeeded:

```python
# Check convergence status first
converged = robjects.r[model_name].rx2('converged')[0]
print(f"Model converged: {converged}")

# If True: your model is fine, no convergence issues
# If False: continue with troubleshooting below
```

## Step 2: Handle Runtime/NaN Errors

**If you get NaN errors or runtime errors** but your data has no actual NaN values, this usually indicates **formula complexity issues**.

**Quick fix - start ultra-simple:**

```python
# Simplest possible GAMLSS model
gamlss_model.fit(
    r_code="""
    model <- gamlss(y ~ age + as.factor(sex),
                   family = NO(),           # Gaussian distribution  
                   method = RS(100))        # No sigma/nu/tau modeling
    """,
    data=df
)
```

**If this simple model works**, gradually add complexity:
1. Add more predictors: `y ~ age + as.factor(sex) + as.factor(site)`
2. Add sigma modeling: `sigma.formula = ~ age`  
3. Switch to SHASH: `family = SHASH()`
4. Add random effects: `re(random=~1|as.factor(site))`

## Step 3: Increase Iterations

**Most convergence issues resolve with more iterations:**

```python
# Default RS(50) is often insufficient
method = RS(300)    # 300 iterations usually sufficient
method = RS(500)    # For stubborn models
```

**Note:** RS(100) means 100 iterations, recommend 300-500 for complex models.

## Step 4: Check Data Scaling Issues

**If model still won't converge, check variable ranges:**

```python
# Essential diagnosis - first examine your data scales  
print(df[['age', 'income', 'brain_volume', 'target_thickness']].describe())
```

**Look for large range differences in the output:**
```
              age       income  brain_volume  target_thickness
count       1000         1000          1000              1000
mean          45        55000          1100             9500
std           15        20000           200             1200
min           18        30000           600             7800
max           80        90000          1400            12000
```

**🚨 Identify the Problem:** Compare the ranges across variables:
- `age`: 18-80 (range ~60)
- `brain_volume`: 600-1400 (range ~800) 
- `income`: 30,000-90,000 (range ~60,000) ← **1000x larger than age**
- `target_thickness`: 7,800-12,000 (range ~4,200) ← **70x larger than age**

**Rule of thumb:** When variables differ by 100x or more in range, scaling is needed for numerical stability.

### Simple Scaling Solution

```python
# Apply scaling based on the actual range differences identified above
df['income_k'] = df['income'] / 1000          # 55,000 → 55 (1000x difference)
df['thickness_h'] = df['target_thickness'] / 100  # 9,500 → 95 (100x difference)

# Keep well-scaled variables unchanged
# age (18-80), brain_volume (600-1400) are already in reasonable ranges

# Verify scaling worked - ranges should now be similar
print(df[['age', 'income_k', 'brain_volume', 'thickness_h']].describe())
```

### ⚠️ Critical: Consistency with Test Data

**Always apply identical scaling to test data:**

```python
# MUST apply same scaling to test data
test_data['income_k'] = test_data['income'] / 1000
test_data['thickness_h'] = test_data['target_thickness'] / 100

# Use scaled variables consistently
x_vals = ['age', 'income_k', 'brain_volume']  
y_val = 'thickness_h'  # Using scaled target

# Fit model with scaled variables
gamlss_model = Gamlss('scaled_model', x_vals, y_val)
gamlss_model.fit(r_code, df)

# Make predictions with same scaled test data  
predictions = gamlss_model.predict_all(test_data, df)
```

### Quick Scaling Guidelines

**First, check your data ranges using `df.describe()`, then apply these rules:**

| Range Difference | Action | Example |
|------------------|--------|---------|
| 1000x+ larger | ÷ 1,000 or ÷ 10,000 | Income: 55,000 → 55 |
| 100x larger | ÷ 100 | Thickness: 9,500 → 95 |
| 10x larger | ÷ 10 | Large measures: 500 → 50 |
| 0.1x smaller | × 10 | Small decimals: 0.05 → 0.5 |
| 0.01x smaller | × 100 | Tiny values: 0.005 → 0.5 |
| Similar range (1-10x) | Keep as-is | Age, education years |
| Binary (0-1) | Keep as-is | Sex, affected status |

**Key principle:** Aim for all variables to have similar ranges (within 10x of each other) for optimal numerical stability.

## Step 5: Model Structure Simplification

**If scaling doesn't help, simplify the model structure:**

### Start Simple, Add Complexity Gradually

```python
# Step 1: Basic linear model (should always converge)
model <- gamlss(y ~ age + as.factor(sex) + as.factor(site),
               family = NO(),
               method = RS(300))

# Step 2: Add smoothing
model <- gamlss(y ~ pvc(age) + as.factor(sex) + as.factor(site),
               family = NO(),
               method = RS(300))

# Step 3: Model sigma parameter  
model <- gamlss(y ~ pvc(age) + as.factor(sex) + as.factor(site),
               sigma.formula = ~ age + as.factor(site),
               family = NO(),
               method = RS(300))

# Step 4: Switch to more complex distribution
model <- gamlss(y ~ pvc(age) + as.factor(sex) + as.factor(site),
               sigma.formula = ~ age + as.factor(site),
               family = SHASH(),
               method = RS(300))

# Step 5: Add random effects (most complex)
model <- gamlss(y ~ pvc(age) + as.factor(sex) + re(random=~1|as.factor(site)),
               sigma.formula = ~ age + as.factor(site),
               family = SHASH(),
               method = RS(300))
```

### Distribution Family Options

**Try simpler distributions if SHASH fails:**

```python
family = NO()     # Gaussian (2 parameters) - start here
family = GA()     # Gamma (2 parameters)  
family = BCT()    # Box-Cox t (4 parameters)
family = SHASH()  # Most flexible (4 parameters) - end goal
```

## Step 6: Algorithm Alternatives

**If RS() doesn't work, try different optimization:**

```python
method = CG(300)        # Cole-Green algorithm
method = mixed(100, 200) # Mixed approach: 100 RS + 200 CG
```

## Data Quality Diagnostics

```python
# Check for data issues that prevent convergence
print("Missing values:", df.isnull().sum().sum())
print("Infinite values:", np.isinf(df.select_dtypes(include=[np.number])).sum().sum())

# Check for extreme outliers
for col in ['age', 'income_k', 'thickness_h']:
    Q1, Q3 = df[col].quantile([0.25, 0.75])
    IQR = Q3 - Q1
    outliers = df[(df[col] < Q1 - 1.5*IQR) | (df[col] > Q3 + 1.5*IQR)]
    print(f"{col}: {len(outliers)} outliers detected")

# Check categorical variable balance
print("\nSite distribution:")
print(df['site'].value_counts())
```

## Complete Troubleshooting Workflow

```python
# 1. Check convergence status
converged = robjects.r[model_name].rx2('converged')[0]
if converged:
    print("✅ Model already converged successfully!")
else:
    print("❌ Convergence failed - starting troubleshooting...")

# 2. Try more iterations first
gamlss_model.fit(
    r_code="""
    model <- gamlss(y ~ pvc(age) + as.factor(sex) + as.factor(site),
                   sigma.formula = ~ age,
                   family = SHASH(),
                   method = RS(500))  # Increased iterations
    """,
    data=df
)

# 3. If still failing, check and fix scaling
print("Checking variable ranges:")
print(df[['age', 'income', 'brain_thickness']].describe())

# Apply scaling based on actual range differences
df['income_k'] = df['income'] / 1000  # If 1000x difference
df['thickness_h'] = df['brain_thickness'] / 100  # If 100x difference

# 4. Apply same scaling to test data consistently
test_data['income_k'] = test_data['income'] / 1000  
test_data['thickness_h'] = test_data['brain_thickness'] / 100

# 5. Retry with scaled data
y_val = 'thickness_h'
x_vals = ['age', 'income_k', 'site']

gamlss_model = Gamlss('final_model', x_vals, y_val)
gamlss_model.fit(
    r_code="""
    model <- gamlss(y ~ pvc(age) + income_k + as.factor(site),
                   sigma.formula = ~ age + as.factor(site),
                   family = SHASH(),
                   method = RS(400))
    """,
    data=df
)

# 6. Verify final convergence
converged = robjects.r[gamlss_model.model_name].rx2('converged')[0]
print(f"Final convergence status: {converged}")
```

## Interpretation with Scaled Variables

### For Predictor Variables (X)
- **No interpretation changes needed** - coefficients adjust automatically
- Statistical significance remains identical

### For Target Variables (Y)
- **Simplest approach:** Keep results in scaled units
- **Label plots clearly:** "Brain Thickness (÷100 units)" or "Thickness /100"

```python
# Z-scores and centiles work normally with scaled data
z_scores = gamlss_model.z_score(test_data, df, func_name='SHASH')  # func_name is optional: the model's family
gamlss_model.plot_z_scores(test_data, df, x_var='age', x_var_name='Age')
# Y-axis shows scaled values - note this in interpretation
```

## When All Else Fails

1. **Test with data subset**: Use 200-500 rows to isolate the issue
2. **Check R console output**: Look for specific error messages
3. **Remove random effects**: Use `as.factor(site)` instead of `re(random=~1|as.factor(site))`
4. **Use simplest distribution**: Start with `family=NO()` 
5. **Remove all smoothing**: Use `age` instead of `pvc(age)`

## Key Success Principles

- **Check convergence status first** - don't assume failure
- **Start simple, add complexity gradually** - build up from basic model
- **Scale based on actual range differences** - apply appropriate scaling ratios
- **Scale consistently** - apply identical transformations to train and test data
- **Use enough iterations** - 300-500 for complex models
- **Keep interpretations in scaled units** - much simpler than converting back

Most convergence issues stem from variable scaling problems or overly complex initial model specifications. Working through this checklist in order -- convergence status, formula complexity, iterations, scaling, model structure, algorithm -- covers the great majority of cases seen in practice.