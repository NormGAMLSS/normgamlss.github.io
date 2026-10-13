# NormGAMLSS Documentation

Welcome to the comprehensive documentation for the NormGAMLSS package! This package provides a powerful framework for fitting and analyzing **Generalized Additive Models for Location, Scale, and Shape** by integrating with the R GAMLSS package through rpy2.

## What is GAMLSS?

GAMLSS extends traditional regression modeling by allowing you to model not just the mean (location) of your response variable, but also its:
- **Scale** (variance/dispersion)
- **Shape** (skewness and kurtosis)

This makes GAMLSS particularly powerful for:
- **Neuroimaging data** with complex distributions
- **Clinical measurements** requiring reference curves
- **Multi-site studies** with heterogeneous populations
- **Outlier detection** and normative modeling

## Key Features

✅ **Flexible Distributions**: 100+ distribution families beyond normal distribution  
✅ **Random Effects**: Proper multi-site modeling with shrinkage  
✅ **Smoothing Terms**: Non-linear relationships with penalized splines  
✅ **Model Comparison**: AIC/BIC comparison with permutation testing  
✅ **Brain Visualization**: Map results to brain surface models  
✅ **Site Transfer**: Adapt models to new sites without retraining  
✅ **Parallel Processing**: Efficient computation for large-scale analysis  

## Why This Python Package?

While GAMLSS is natively available in R, this Python package provides:
- **Seamless Integration**: Work in Python while leveraging R's GAMLSS power
- **Automated Workflows**: Batch processing, caching, and model management
- **Neuroimaging Focus**: Brain visualization and multi-site transfer tools
- **Production Ready**: Save/load models, robust error handling

## Quick Example

```python
from normgamlss import Gamlss

# Fit a GAMLSS model
model = Gamlss('my_model', ['age', 'sex', 'site'], 'brain_volume')
model.fit(r_code="""
    model <- gamlss(y ~ pvc(age, by = as.factor(sex)) + re(random=~1|site),
                   sigma.formula = ~ pvc(age) + re(random=~1|site), 
                   family = SHASH(), method = RS(50))
""", data=train_data)

# Get z-scores for outlier detection
z_scores = model.z_score(test_data, train_data)

# Create reference curves
model.plot_centiles(test_data, train_data, centiles=[3, 10, 50, 90, 97])
```

## Documentation Structure

This documentation follows a practical workflow:

1. **Setup**: Installation and data preparation
2. **Basic Modeling**: Fit your first GAMLSS model  
3. **Evaluation**: Assess model quality and interpret results
4. **Advanced Features**: Model comparison, permutation testing
5. **Applications**: Brain visualization, multi-site studies
6. **Reference**: Complete syntax guide and troubleshooting

## Getting Started

Ready to dive in? Start with the [Installation Guide](installation.md) or jump straight to the [Quick Start](quick_start.md) for a 5-minute tutorial.

## Acknowledgments

This package builds on the excellent work of:
- **R GAMLSS Team**: [gamlss.com](https://www.gamlss.com/) - The original GAMLSS implementation
- **rpy2 Project**: [rpy2.github.io](https://rpy2.github.io/) - R integration for Python 

---

*This documentation covers the Python wrapper for GAMLSS. For comprehensive theoretical background, statistical details, and additional examples, please refer to the [official R GAMLSS documentation](https://www.gamlss.com/).*