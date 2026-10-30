# Comprehensive Model Diagnostics Interpretation

This chapter provides detailed guidance for interpreting GAMLSS model outputs, summary statistics, and diagnostic plots to assess model quality and validity.

## Understanding Model Summary Output

The GAMLSS model summary provides comprehensive information about your fitted model. Using the default `type="vcov"`, the variance covariance matrix is calculated using the inverse of the numerical second derivatives of the observed information matrix, which is more reliable since it takes into account the inter-correlation between all parameters. If this fails, R warns "summary: vcov has failed, option qr is used instead" and uses the QR decomposition of each parameter's fit instead.

### Family and Call Information

**Family**: Specifies the distribution used (e.g., "SHASH", "NO", "GA")
- Indicates the flexibility of your model
- SHASH allows for skewness and kurtosis modeling
- Simpler families like NO (Normal) assume symmetric distributions

**Call**: Shows the complete model specification
- Documents the exact formulas used for each parameter
- Includes data source and fitting method
- Essential for reproducibility

### Fitting Method

**Common Methods**:
- **RS()**: Rigby-Stasinopoulos algorithm (the default, stable)
- **CG()**: Cole-Green algorithm (uses the cross-derivatives between parameters)
- **mixed(n1, n2)**: n1 RS iterations, then up to n2 CG iterations

If the model has not converged, try more RS iterations or `mixed(10, 100)`; neither always
wins (see the [FAQ](faq)).

The number in parentheses (e.g., RS(50)) indicates maximum iterations. More iterations generally improve convergence but increase computation time.

### Parameter-Specific Sections

#### Link Functions
Each parameter uses a specific transformation:
- **Identity link**: Parameter equals the linear predictor directly
- **Log link**: Ensures positive parameter values (common for sigma, nu, tau)
- **Logit link**: Constrains parameters to (0,1) range

**Critical for interpretation**: When making predictions, you must apply the inverse link function to get actual parameter values.

#### Coefficient Interpretation

For each parameter section, examine:

**Estimate**: The fitted coefficient value
- For identity link: Direct effect on the parameter
- For log link: Effect on the log of the parameter

**Standard Error**: Uncertainty measure accounting for inter-correlation between all distributional parameters
- Smaller values indicate more precise estimates
- Used to calculate confidence intervals

**t-value**: Estimate divided by standard error
- Measures how many standard errors the estimate is from zero
- Values > 2 or < -2 typically indicate significance

**p-value (Pr(>|t|))**: Probability of a |t| at least this large if the true coefficient were 0
- **< 0.001**: Highly significant (***)
- **< 0.01**: Very significant (**)
- **< 0.05**: Significant (*)
- **> 0.05**: Not statistically significant

#### Significance Codes
The asterisk system provides quick visual assessment:
```
'***' p < 0.001 (highly significant)
'**'  p < 0.01  (very significant)  
'*'   p < 0.05  (significant)
'.'   p < 0.1   (marginally significant)
' '   p ≥ 0.1   (not significant)
```

**Action items for non-significant predictors**:
- Consider removing if p > 0.05 and theoretically unimportant
- Retain if theoretically crucial regardless of significance
- Use model comparison to assess impact of removal

### Model Fit Statistics

#### Degrees of Freedom
**Degrees of Freedom for fit**: Effective number of parameters estimated
- Accounts for smoothing complexity
- Higher values indicate more complex models

**Residual Degrees of Freedom**: Sample size minus effective parameters
- `df_residual = n_observations - df_fit`
- Very low residual df suggests potential overfitting

#### Information Criteria
**Global Deviance**: Measure of model fit (lower is better)
- Comparable across models with same data
- Shows fitting progression across iterations

**AIC (Akaike Information Criterion)**: Balances fit and complexity
- `AIC = Global Deviance + 2 × df_fit`
- Lower values indicate better models
- Differences > 2 suggest meaningful improvement

**SBC/BIC (Schwarz/Bayesian Information Criterion)**: More conservative than AIC
- `BIC = Global Deviance + log(n) × df_fit`
- Penalizes complexity more heavily than AIC
- Preferred when sample size is large

### Notes and Warnings

**Smoothing Terms Warning**: 
> "NOTE: Additive smoothing terms exist in the formulas:
> i) Std. Error for smoothers are for the linear effect only.
> ii) Std. Error for the linear terms maybe are not accurate."

When R has fallen back to the QR method (see above), the second line reads "ii) Std. Error for
the linear terms may not be reliable."

This means:
- Standard errors may not capture full uncertainty for smooth terms
- p-values for linear terms may be unreliable when smoothing is present
- Focus on overall model diagnostics rather than individual p-values
