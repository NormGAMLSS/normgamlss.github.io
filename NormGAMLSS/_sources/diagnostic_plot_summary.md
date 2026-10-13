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
- **S-shaped curve**: Indicates skewness
- **Curved at ends**: Heavy or light tails
- **Systematic deviations**: Non-normality

**Interpretation guidance**: Departure from normality may be important depending on analysis purpose - especially critical when estimating centiles in the tails

## Summary Statistics for Residuals
When running diagnostic plots, GAMLSS provides summary statistics for the quantile residuals:

### Mean
**Ideal value**: ≈ 0
**Interpretation**: 
- Values close to zero indicate unbiased model
- Systematic deviation suggests model bias

### Variance
**Ideal value**: ≈ 1
**Interpretation**:
- Values close to 1 indicate proper scaling
- Much larger values suggest underdispersion modeling
- Much smaller values suggest overdispersion

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
- Values < 0.95 may indicate normality violations
- Lower values suggest need for different distribution

## Model Assessment Guidelines
### When the Model is Performing Well
**Good signs**:
- All diagnostic plots show random scatter
- Summary statistics near ideal values
- Residual variance ≈ 1 and mean ≈ 0
- Filliben correlation > 0.99
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
3. Try tensor product smooths `te(x1, x2)`

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
- Check lambda values (indicates shrinkage)
- Compare degrees of freedom to number of groups
- Verify random effects are actually being applied

### Multi-parameter Modeling
When modeling multiple parameters (sigma, nu, tau):
- Each parameter should be assessed separately
- Consider interaction between parameter modeling
- Simpler is often better for less important parameters
