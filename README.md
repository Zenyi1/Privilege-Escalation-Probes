# Privilege Escalation Probes

A machine learning project that uses linear probing techniques to detect privilege escalation attempts in command-line inputs by analyzing the internal representations of GPT-2.

## Overview

This project trains linear classifiers (probes) on GPT-2's internal activations to identify potentially dangerous commands that could be used for privilege escalation. The approach leverages the hypothesis that language models develop internal representations that capture semantic concepts like "dangerous" or "privileged" operations.

## Features

- **Multi-layer Analysis**: Extracts and analyzes activations from different layers of GPT-2
- **Data Leakage Prevention**: Ensures no command templates appear in both training and test sets
- **Comprehensive Evaluation**: Includes accuracy, F1-score, ROC-AUC, and precision-recall metrics
- **Category Analysis**: Breaks down performance by command categories
- **Error Analysis**: Identifies and analyzes misclassified examples
- **Baseline Comparison**: Compares against keyword-based detection methods

## Requirements

```bash
pip install transformers torch transformer-lens datasets scikit-learn matplotlib seaborn polars pandas numpy
```

## Data Format

The script expects data in JSONL format with the following structure:

```json
{"command": "sudo apt update", "label": true, "category": "system_admin", "obfuscation_level": 0}
{"command": "ls -la", "label": false, "category": "file_operations", "obfuscation_level": 0}
```

### Required Fields:
- `command`: The command-line input to analyze
- `label`: Boolean indicating if this is a privilege escalation attempt
- `category`: Category of the command (e.g., "system_admin", "file_operations")
- `obfuscation_level`: Integer indicating level of command obfuscation

## Usage

1. **Prepare your data**: Place your JSONL file at `/content/sample_data/sample_data.jsonl`

2. **Run the analysis**:
```python
python privilege_escalation_probes.py
```

3. **View results**: The script will output:
   - Performance metrics across different GPT-2 layers
   - Best performing layer identification
   - Category-wise accuracy breakdown
   - Confusion matrices and classification reports
   - Error analysis with misclassified examples

## Key Functions

### `extract_activations(commands, layer_idx=-1, activation_type="resid_post")`
Extracts internal representations from specified GPT-2 layer for given commands.

### `train_and_evaluate_probe_no_leakage(activations, labels, commands)`
Trains linear probe while preventing data leakage by ensuring command templates don't appear in both train/test sets.

### `analyze_by_category(test_df, results)`
Provides detailed performance breakdown by command categories.

### `keyword_baseline(df)`
Implements simple keyword-based baseline using dangerous command patterns.

## Model Architecture

- **Base Model**: GPT-2 (117M parameters)
- **Probe**: Logistic Regression classifier
- **Input Processing**: Commands tokenized and truncated to 128 tokens
- **Activation Pooling**: Mean pooling across sequence length
- **Layers Analyzed**: 0, 3, 6, 9, 11 (configurable)

## Output Metrics

The script provides comprehensive evaluation including:

- **Accuracy**: Overall classification accuracy
- **Precision/Recall/F1**: For privilege escalation detection
- **ROC-AUC**: Area under ROC curve
- **PR-AUC**: Area under precision-recall curve
- **Calibration**: Average predicted probabilities by class
- **Category Performance**: Accuracy breakdown by command type

## Example Output

```
Dataset overview:
Total samples: 5000
Positive samples: 2500
Negative samples: 2500

Best performing layer: layer_9
Test accuracy: 0.847

Category-wise analysis:
                Accuracy  Count  Avg_Probability
system_admin       0.923     65            0.756
file_operations    0.834    120            0.432
network_ops        0.778     45            0.567
```

## Data Leakage Prevention

The script implements template-based splitting to prevent data leakage:

1. Commands are converted to templates (paths → `/PATH`, numbers → `<NUM>`)
2. Templates are split between train/test sets
3. No template appears in both training and testing data

## Customization

### Analyzing Different Layers
```python
layers_to_analyze = [0, 2, 4, 6, 8, 10]  # Customize layers
all_layer_activations = extract_multiple_layers(all_commands, layers_to_analyze)
```

### Adjusting Model Parameters
```python
# In extract_activations function
max_length=256,  # Increase for longer commands
batch_size=16,   # Adjust based on GPU memory
```

### Custom Baseline Keywords
```python
dangerous_keywords = ['sudo', 'su', 'chmod', 'your_custom_keywords']
```

## Limitations

- Requires GPU for efficient processing (though CPU compatible)
- Limited to GPT-2 architecture (easily extensible to other models)
- Performance depends on quality and diversity of training data
- May not generalize to heavily obfuscated or novel attack patterns

## Future Improvements

- Support for larger language models (GPT-3, GPT-4)
- Multi-modal analysis combining command structure and context
- Active learning for iterative model improvement
- Real-time deployment capabilities
- Support for other activation types (attention patterns, MLP outputs)
