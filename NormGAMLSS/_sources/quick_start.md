# Quick Start Guide

Get up and running with GAMLSS in 5 minutes! This tutorial walks through a complete example from data preparation to model evaluation.

## 5-Minute Tutorial

### Step 1: Setup and Import

```python
import numpy as np
import pandas as pd
import rpy2.robjects as robjects
from rpy2.robjects.packages import importr

# Import GAMLSS functions
from normgamlss import Gamlss
from normgamlss.utils import *
from normgamlss.model_utils import *

# Setup R environment
gamlss = importr('gamlss')
base = importr('base')
stats = importr('stats')
```

### Step 2: Create Example Data

```python
# Generate sample neuroimaging data
np.random.seed(42)
n_samples = 500

# Create training data
train_data = pd.DataFrame({
    'age': np.random.randint(18, 80, n_samples),
    'sex': np.random.choice([0, 1], n_samples),
    'site': np.random.choice(['Site_A', 'Site_B', 'Site_C'], n_samples),
    'brain_volume': np.random.normal(1000, 100, n_samples)
})

# Add realistic age and sex effects: volume declines steeply through early
# adulthood and then flattens out at older ages (non-linear), with a
# near-parallel sex offset -- the shape seen in real neuroimaging data
train_data['brain_volume'] += (
    200 * np.exp(-train_data['age'] / 30) +  # Non-linear age decline (steep early, flat late)
    50 * train_data['sex'] +                 # Sex difference
    np.random.normal(0, 50, n_samples)       # Noise
)

# Create test data with some outliers
test_data = train_data.copy().sample(200).reset_index(drop=True)
outlier_idx = np.random.choice(200, 20, replace=False)
test_data.loc[outlier_idx, 'brain_volume'] += np.random.normal(0, 200, 20)
test_data['condition'] = 'healthy'
test_data.loc[outlier_idx, 'condition'] = 'patient'

print("✅ Sample data created")
print(f"Training: {len(train_data)} samples")
print(f"Testing: {len(test_data)} samples (20 outliers)")
```

### Step 3: Fit Your First GAMLSS Model

```python
# Define model variables
model_name = 'quick_start_model'
x_vals = ['age', 'sex', 'site']  
y_val = 'brain_volume'

# Create GAMLSS model instance
model = Gamlss(model_name, x_vals, y_val)

# Fit the model with random effects for site
model.fit(
    r_code="""
    model <- gamlss(y ~ pvc(age, by = as.factor(sex)) + re(random=~1|as.factor(site)),
        sigma.formula = ~ pvc(age) + re(random=~1|as.factor(site)),
        nu.formula = ~ 1,
        tau.formula = ~ 1,
        family = SHASH(),
        method = RS(50)
    )
    """,
    data=train_data
)

print("✅ Model fitted successfully!")
```

### Step 4: Check Model Quality

```python
# Get model summary
summary = base.summary(robjects.r[model_name])
print("📊 Model Summary:")
print(summary)

# Quick diagnostic plots
model.plot_diagnostics()
print("✅ Diagnostic plots created")

# Evaluate model performance on the healthy subjects only
# ('healthy' labels count as healthy; 'patient' rows are left out)
results = model.evaluate(test_data, train_data, affected=test_data['condition'])
print("\n📈 Model Evaluation:")
print(results)
```

### Step 5: Detect Outliers with Z-Scores

```python
# Calculate z-scores for outlier detection
z_scores = model.z_score(test_data, train_data, func_name='SHASH')  # func_name is optional: the model's family

# Plot z-scores colored by condition
model.plot_z_scores(
    test_data, train_data, 
    affected=test_data['condition'],
    x_var='age', 
    title="Z-scores: Outlier Detection"
)

# Count outliers (|z| > 1.96)
outliers = np.abs(z_scores) > 1.96
print(f"\n🎯 Outlier Detection Results:")
print(f"Statistical outliers detected: {outliers.sum()}/{len(z_scores)}")
print(f"True patients in data: {(test_data['condition'] == 'patient').sum()}")
```

### Step 6: Create Reference Curves  

```python
# Generate centile curves
model.plot_centiles(
    test_data, train_data,
    centiles=[3, 10, 25, 50, 75, 90, 97],
    variables_to_split={'sex': [0, 1]},
    x_axis={'age': 'Age (years)'},
    affected=test_data['condition']
)

print("✅ Reference curves created")
```

### Step 7: Save Your Model

```python
# Save model for later use
model.save_model("quick_start_model.rds")
print("💾 Model saved as 'quick_start_model.rds'")

# Load it back (demonstration)
loaded_model = Gamlss.load_model("quick_start_model.rds")
print("📂 Model loaded successfully!")
```

## What You Just Accomplished

🎉 **Congratulations!** In just a few minutes, you've:

✅ **Fitted a sophisticated GAMLSS model** with:
- Smooth age curves that differ by sex
- Random effects for multi-site data
- Flexible SHASH distribution

✅ **Assessed model quality** through:
- Diagnostic plots and summary statistics
- Comprehensive evaluation metrics

✅ **Detected outliers** using:
- Z-score calculations
- Visual plots with condition labeling

✅ **Created reference curves** showing:
- Age-specific centiles by sex
- Clinical interpretation ranges

✅ **Managed your model** with:
- Save/load functionality for reproducibility

## Understanding the Results

### Z-Scores
- **|Z| < 1.96**: Normal range (95% of healthy population)
- **|Z| > 1.96**: Statistical outliers  
- Values closer to ±4 indicate more extreme deviations

### Centile Curves
- **50th centile**: Median/expected value for age/sex
- **3rd-97th centiles**: Normal range boundaries
- Points outside these ranges warrant clinical attention

### Model Evaluation Metrics
- **Z_Scores_Mean ≈ 0**: Unbiased model
- **Z_Scores_SD ≈ 1**: Proper scaling
- **Logarithmic_Score**: Higher = better probabilistic fit

## Next Steps

Now that you've seen GAMLSS in action, explore these areas:

### 📚 **Learn More**
- [Data Setup Theory](data_and_setup_theory.md) - R integration and external packages
- [Formula Syntax](formulas_and_syntax.md) - Advanced model specification  
- [Model Diagnostics](notebooks/model_diagnostics.ipynb) - Assess model quality

### 🔬 **Advanced Features**  
- [Model Comparison](notebooks/multiple_formula_comparison.ipynb) - Compare different specifications
- [Permutation Testing](notebooks/permutation_testing.ipynb) - Statistical significance
- [Brain Visualization](notebooks/brain_surface_visualization.ipynb) - Map results to brain surfaces
- [Multi-site Transfer](notebooks/multisite_transfer.ipynb) - Adapt models to new sites

### 🛠️ **Real Applications**
- Replace sample data with your neuroimaging datasets
- Experiment with different distribution families (NO, GG, GA)
- Add more predictors (scanner type, acquisition parameters)
- Compare models with and without random effects

## Common Next Questions

**Q: How do I use my own data?**  
A: Replace the sample data creation with `pd.read_csv('your_data.csv')`. Ensure you have columns for predictors and outcome variable.

**Q: My model won't converge - what should I do?**  
A: Start with a simpler model (fewer predictors), increase iterations `RS(100)`, or check the [Troubleshooting Guide](troubleshooting.md).

**Q: How do I interpret the diagnostic plots?**  
A: See the [Model Diagnostics](notebooks/model_diagnostics.ipynb) notebook for detailed interpretation guidelines.

**Q: Can I use different distributions?**  
A: Yes! Try `family = NO()` for normal, `family = GA()` for gamma, or `family = GG()` for generalized gamma. Pass the same name as `func_name` (e.g. `func_name='NO'`) to `z_score`, `evaluate`, `centiles` and the plotting methods, which default to `'SHASH'`.

---

Ready to dive deeper? The full documentation provides comprehensive coverage of every feature, complete with real-world examples and best practices.