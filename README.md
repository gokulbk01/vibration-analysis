# Vibration Analysis Library

A Python library for preprocessing and analyzing vibration data from rotating and reciprocating assets.

## Features

- **Data Validation**: Load and validate vibration data with automatic type conversion
- **Threshold Filtering**: Filter data based on minimum vibration threshold
- **Outlier Removal**: Smart outlier detection that preserves consecutive high-value sequences
- **Moving Average**: Configurable moving average calculation
- **Reference Points**: Automatic extraction of early, middle, and late period reference points
- **Deviation Analysis**: Calculate deviations and IQR bounds for anomaly detection

## Installation

### From PyPI (once published)
```bash
pip install vibration-analysis
```

### From Source
```bash
git clone https://github.com/gokulbk01/vibration-analysis.git
cd vibration-analysis
pip install -e .
```

### For Development
```bash
pip install -e ".[dev]"
```

## Quick Start

```python
import pandas as pd
from vibration_analysis import VibrationPreprocessor

# Load your data
data = pd.read_csv('vibration_data.csv')

# Initialize preprocessor with custom parameters
preprocessor = VibrationPreprocessor(
    moving_average_window=30,  # Window size for moving average
    min_tve=0.065,             # Minimum threshold vibration energy
    outlier_std_threshold=8.0   # Standard deviation multiplier for outliers
)

# Run complete preprocessing pipeline
results = preprocessor.preprocess(data)

# Access processed data
processed_data = results['processed_data']
reference_points = results['reference_points']
iqr_bounds = results['iqr_bounds']

# View reference points
for name, point in reference_points.items():
    print(f"{name}: MA={point['moving_average']:.4f}, Month={point['month']}")

# View IQR statistics
print(f"Upper Bound: {iqr_bounds['upper_bound']:.4f}")
```

## Detailed Usage

### Basic Preprocessing

```python
from vibration_analysis import VibrationPreprocessor

# Initialize with defaults for rotating assets
preprocessor = VibrationPreprocessor(
    moving_average_window=30,
    min_tve=0.065
)

# For reciprocating assets
preprocessor_recip = VibrationPreprocessor(
    moving_average_window=50,
    min_tve=0.15
)

# Process data
results = preprocessor.preprocess(data)
```

### Step-by-Step Processing

```python
# Initialize preprocessor
preprocessor = VibrationPreprocessor(moving_average_window=30)

# Step 1: Validate data
validated_data = preprocessor.load_and_validate_data(data)

# Step 2: Filter by threshold
filtered_data = preprocessor.filter_by_threshold(validated_data)

# Step 3: Remove outliers
cleaned_data, removed_outliers, threshold = preprocessor.remove_outliers(filtered_data)

# Step 4: Calculate moving average
ma_data = preprocessor.calculate_moving_average(cleaned_data)

# Step 5: Get reference points
reference_points = preprocessor.get_reference_points(ma_data)

# Step 6: Calculate deviations
processed_data = preprocessor.calculate_deviations(ma_data)

# Step 7: Calculate IQR bounds
iqr_bounds = preprocessor.calculate_iqr_bounds(processed_data)
```

### Custom Column Names

```python
# If your data has different column names
results = preprocessor.preprocess(
    data,
    datetime_col='timestamp',
    broadband_col='vibration_value'
)
```

## Results Structure

The `preprocess()` method returns a dictionary with the following keys:

- **`processed_data`**: DataFrame with original data plus:
  - `Moving_Average`: Calculated moving average
  - `Deviation`: Difference between broadband and moving average

- **`removed_outliers`**: DataFrame containing outlier points that were removed

- **`outlier_threshold`**: Float value of the threshold used for outlier detection

- **`reference_points`**: Dictionary with three reference periods:
  ```python
  {
      'Early_Period': {
          'datetime': timestamp,
          'moving_average': float,
          'broadband': float,
          'month': str,
          'index': int,
          'days_from_start': int
      },
      'Middle_Period': {...},
      'Late_Period': {...}
  }
  ```

- **`iqr_bounds`**: Dictionary with IQR statistics:
  ```python
  {
      'Q1': float,
      'Q3': float,
      'IQR': float,
      'upper_bound': float
  }
  ```

- **`config`**: Dictionary with preprocessor configuration

## Parameters

### VibrationPreprocessor

- **`moving_average_window`** (int, default=30): Window size for calculating moving average
  - Rotating assets: typically 30
  - Reciprocating assets: typically 50

- **`min_tve`** (float, default=0.065): Minimum threshold vibration energy
  - Rotating assets: typically 0.065
  - Reciprocating assets: typically 0.15

- **`outlier_std_threshold`** (float, default=8.0): Standard deviation multiplier for outlier detection
  - Higher values make outlier detection less sensitive

## Requirements

- Python >= 3.7
- pandas >= 1.3.0
- numpy >= 1.20.0

## Example: Processing Multiple Files

```python
import os
from pathlib import Path
from vibration_analysis import VibrationPreprocessor

# Initialize preprocessor
preprocessor = VibrationPreprocessor(moving_average_window=30, min_tve=0.065)

# Process all CSV files in a directory
data_folder = Path('vibration_data')
all_results = {}

for csv_file in data_folder.glob('*.csv'):
    data = pd.read_csv(csv_file)
    results = preprocessor.preprocess(data)
    all_results[csv_file.name] = results
    
    # Save processed data
    output_file = f"processed_{csv_file.name}"
    results['processed_data'].to_csv(output_file, index=False)
    
    print(f"Processed {csv_file.name}")
    print(f"  Reference points: {len(results['reference_points'])}")
    print(f"  Upper bound: {results['iqr_bounds']['upper_bound']:.4f}")
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details.

## Support

For issues, questions, or contributions, please visit:
https://github.com/gokulbk01/vibration-analysis

## Changelog

### Version 0.1.0 (Initial Release)
- Basic preprocessing functionality
- Moving average calculation
- Reference point extraction
- IQR-based deviation analysis
- Outlier detection and removal
