# Diagnostic Plots Interpretation
GAMLSS diagnostic plots check the normalized (randomized for discrete distributions) quantile residuals through four standard plots: residuals vs fitted values, residuals vs index, density plot, and normal Q-Q plot.

## 1. Against Fitted Values Plot
**Purpose**: Assess heteroscedasticity and non-linearity

**Ideal appearance**:
- Random scatter around horizontal line at zero
- Constant spread across fitted value range
- No systematic patterns or curves

**Problematic patterns**:
- **Funnel shape**: Indicates heteroscedasticity (changing variance)
- **Curved patterns**: Suggests missing non-linear terms
- **Systematic trends**: May indicate model misspecification

**Actions for problems**:
- Model sigma parameter with covariates for heteroscedasticity
- Add polynomial or smoothing terms for non-linearity
- Consider data transformation

**Also plot the residuals against age**: in normative models the mean is often U-shaped in age,
so a spread that grows with age may not show against the fitted values at all. In a toy test
with sigma left unmodelled, the funnel was invisible here but clear against age:
```python
r = model.get_residuals()          # same rows and order as the training data
plt.scatter(train_data['age'], r, s=5); plt.axhline(0, ls='--')
```

## 2. Against Index Plot
**Purpose**: Identify temporal patterns or systematic data collection issues

**Ideal appearance**:
- Random scatter with no discernible patterns
- No trending or cyclical behavior
- Constant variance across observation order

**Problematic patterns**:
- **Time trends**: Systematic increase/decrease
- **Cyclical patterns**: Regular oscillations
- **Clustering**: Groups of similar residuals

The order of the rows matters here. If the data are sorted by site, steps or blocks in this
plot mean a site effect is missing: add `re(random=~1|as.factor(site))`.

**Actions for problems**:
- Include time variables in model
- Check data collection procedures
- Consider autocorrelation structure

## 3. Density Estimate Plot
**Purpose**: Visualize overall distribution of residuals

**Ideal appearance**:
- Symmetric, unimodal distribution
- Centered approximately at zero
- Bell-shaped curve resembling normal distribution

**Problematic patterns**:
- **Skewness**: Asymmetric distribution
- **Multiple modes**: Bimodal or multimodal distributions  
- **Heavy tails**: Extreme values more common than expected

**Actions for problems**:
- Try more flexible distribution families (SHASH, GG)
- Check for outliers or data quality issues
- Consider mixture distributions

## 4. Normal Q-Q Plot
**Purpose**: Assess whether residuals follow normal distribution by comparing quantiles

**Ideal appearance**:
- Points closely follow diagonal reference line
- Minimal deviation especially in central region
- Slight deviation at extremes is acceptable

**Problematic patterns**:
- **Bow or banana shape** (both ends bend the same way): skewness. Both ends above the line
  means right skew, both below means left skew
- **S shape** (the ends bend in opposite directions): heavy tails when the lowest points fall
  below the line and the highest above it; light tails when it is the other way round
- **Systematic deviations**: Non-normality

**Interpretation guidance**: Departure from normality may be important depending on analysis purpose - especially critical when estimating centiles in the tails

## Summary Statistics for Residuals
When running diagnostic plots, GAMLSS provides summary statistics for the quantile residuals.

On the training data the mean and the variance are close to 0 and 1 almost by construction,
even for a poor model: in toy tests with a missed age curve, unmodelled sigma, skewed or
heavy-tailed data they stayed at 0.00 and 1.00. There, the plots, the skewness, the kurtosis
and the Filliben coefficient show the problems. Mean and variance are most useful on new data,
for example the z-scores of a test set.

### Mean
**Ideal value**: ≈ 0

**Interpretation**:
- Values close to zero indicate unbiased model
- Systematic deviation suggests model bias

### Variance
**Ideal value**: ≈ 1

**Interpretation**:
- Values close to 1 indicate proper scaling
- Above 1: the model's spread is too small (the data vary more than modelled)
- Below 1: the model's spread is too large

### Coefficient of Skewness
**Ideal value**: ≈ 0

**Interpretation**:
- Positive values: Right-skewed residuals
- Negative values: Left-skewed residuals  
- Values > |0.5| may indicate distribution issues

### Coefficient of Kurtosis
**Ideal value**: ≈ 3 (normal distribution)

*Note:* this is R's plain kurtosis. The `Kurtosis` metric from `evaluate()` is *excess*
kurtosis (kurtosis − 3), so its target is ≈ 0.

**Interpretation**:
- Values > 3: Heavy-tailed (more extreme values)
- Values < 3: Light-tailed (fewer extreme values)
- Significant departure suggests distribution choice issues

### Filliben Correlation Coefficient
**Ideal value**: ≈ 1 (close to 1)

**Interpretation**:
- Measures correlation between residuals and normal quantiles
- What counts as low depends on the sample size. With a correct model, 95% of simulated
  samples gave at least 0.977 (n = 50), 0.988 (n = 100), 0.997 (n = 500) and 0.9985
  (n = 1000). Values below these suggest a different distribution: a fixed cut-off such as
  0.95 is too lax (in toy tests with n = 1000, heavy-tailed data gave 0.954 and light-tailed
  data 0.978)

## Model Assessment Guidelines
### When the Model is Performing Well
**Good signs**:
- All diagnostic plots show random scatter
- Summary statistics near ideal values
- Residual variance ≈ 1 and mean ≈ 0
- Filliben correlation above the values for your sample size (see above)
- No systematic patterns in any plot

### When the Model Needs Improvement
**Warning signs**:
- Clear patterns in residual plots
- Summary statistics far from ideal values
- Systematic deviations in Q-Q plot
- Very low or high residual variance
- Significant skewness or kurtosis

### Action Steps for Model Issues
**For heteroscedasticity**:
1. Model sigma parameter with covariates
2. Try different link functions
3. Consider data transformation

**For non-linearity**:
1. Add smoothing terms `pvc(x)` or `cs(x)`
2. Include polynomial terms

**For distribution issues**:
1. Try more flexible families (SHASH, GG, BCT)
2. Check for outliers in data
3. Consider zero-inflated or mixture distributions

**For temporal patterns**:
1. Include time variables in model
2. Add lagged variables
3. Consider time series modeling approaches

## Advanced Diagnostic Considerations
### Smoothing Terms
When using smoothing terms like `pvc(age)`:
- Standard errors are approximate
- Focus on overall model fit rather than individual significance
- Use model comparison for term selection

### Random Effects Assessment
For models with random effects:
- Check that the term matters: compare AIC/BIC of the model with and without it
- Compare degrees of freedom to number of groups
- Verify random effects are actually being applied

### Multi-parameter Modeling
When modeling multiple parameters (sigma, nu, tau):
- Each parameter should be assessed separately
- Consider interaction between parameter modeling
- Simpler is often better for less important parameters
