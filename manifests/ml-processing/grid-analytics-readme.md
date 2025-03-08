# Grid Station Analytics & ML

This document provides an overview of the mathematical and analytical concepts implemented in the Grid Station monitoring system.

## Overview

The Grid Station monitoring system collects time-series data from electrical grid equipment via IEC61850 protocol, processes this data through various analytical methods, and provides insights through visualization and alerts. The system demonstrates how modern analytics and ML techniques can be applied to industrial control systems.

## Core Mathematical Concepts

### Time Series Analysis

The system processes temporal data with several techniques:

- **Moving Averages**: Calculating trends over sliding time windows
- **Seasonality Detection**: Identifying and accounting for day/night patterns and hourly variations
- **Trend Analysis**: Computing directional changes in power consumption metrics

### Anomaly Detection

The `AnomalyDetector` class implements statistical anomaly detection:

```python
z_score = abs((value - baseline['mean']) / baseline['std'])
if z_score > 3:  # Beyond 3 standard deviations
    anomalies[feature] = { ... }
```

Key components:
- Z-Score calculations to identify statistical outliers
- Baseline statistics (mean, standard deviation) for each monitored metric
- Multi-threshold approach (medium vs. high severity)
- Domain-specific rules for critical parameters like frequency and temperature

### Predictive Models

The `PowerPredictor` class implements forecasting:

```python
# Trend calculation from recent values
trend = (recent_values[-1] - recent_values[0]) / len(recent_values)

# Prediction combines historical patterns with recent trend
prediction = baseline + (trend * i * 12)  # Extrapolate trend
```

The system uses:
- Historical pattern matching (finding similar time periods)
- Linear trend extrapolation
- Time-of-day awareness for predictions

### Equipment Health Monitoring

The `EquipmentHealthMonitor` tracks equipment degradation:

- Linear degradation models based on operational parameters
- Temperature-accelerated degradation for transformers
- Operation count tracking for circuit breakers
- Impedance and harmonic distortion monitoring for transmission lines
- Classification of maintenance urgency (low, medium, high)

### Classification System

The system classifies grid states across multiple dimensions:

- **Power Flow**: normal, high_load, low_load
- **Stability**: stable, unstable
- **Power Quality**: good, fair, poor
- **Equipment Status**: healthy, maintenance, critical
- **Overall System State**: normal, warning, alert

## ML Implementation

The system is designed to support LSTM (Long Short-Term Memory) neural networks for time series prediction, as specified in `ml-processing-config.yaml`:

```yaml
model:
  type: lstm
  parameters:
    hidden_size: 128
    num_layers: 2
    dropout: 0.2
training:
  epochs: 50
  batch_size: 32
  learning_rate: 0.001
data:
  window_size: 24
  prediction_horizon: 6
```

Key ML concepts:
- **LSTM Architecture**: Specialized recurrent neural network for sequence learning
- **Sliding Window Approach**: Using 24 time steps to predict the next 6
- **Regularization**: Dropout of 0.2 to prevent overfitting

## Data Processing Pipeline

1. **Data Collection**: Raw measurements from IEC61850 simulator stored in `measurements.csv`
2. **Feature Extraction**: Key electrical parameters extracted and normalized
3. **Anomaly Detection**: Statistical analysis identifies outliers
4. **Prediction**: Time series forecasting for future power requirements
5. **Classification**: Multi-parameter classification of system state
6. **Health Monitoring**: Degradation tracking for key equipment
7. **Visualization**: Prometheus metrics and Grafana dashboards
8. **Alerting**: Threshold-based alerting for critical conditions

## Key Analytics Outputs

The system produces several analytical outputs:

1. **Power Consumption Prediction**: 6-hour ahead load forecasting
2. **Anomaly Alerts**: Notifications of unusual parameter values
3. **Equipment Health Scores**: Percentage-based health metrics for key components
4. **Maintenance Recommendations**: Urgency classification for equipment maintenance
5. **System State Classification**: Overall assessment of grid status
6. **Power Quality Analysis**: Metrics on frequency stability and harmonic distortion

## Integration Points

The analytics system integrates with:

- **Prometheus**: For metrics collection and storage
- **Grafana**: For data visualization and dashboarding
- **OPC UA Server**: For exposing processed data to industrial systems
- **Kubernetes**: For deployment and scaling of analytics components

## Extending the Analytics

The system can be extended in several ways:

1. **Full LSTM Implementation**: Replace the simulated ML with actual PyTorch LSTM models
2. **Reinforcement Learning**: Add RL-based control optimization
3. **Multivariate Analysis**: Enhance correlation detection between parameters
4. **Ensemble Methods**: Combine multiple prediction models for improved accuracy
5. **Bayesian Methods**: Add uncertainty quantification to predictions

## Conclusion

The Grid Station monitoring system demonstrates how modern analytics and ML techniques can be applied to industrial control systems. The combination of statistical analysis, time series prediction, and equipment health monitoring provides comprehensive visibility into grid operations.
