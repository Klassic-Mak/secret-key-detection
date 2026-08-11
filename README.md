# Secret Detection Project

This project focuses on building a deep learning model to detect secrets within text data. The model is trained to classify text snippets as either 'Safe' (no secret detected) or 'Secret' (a secret is present).

## Table of Contents

1.  [Project Overview](#project-overview)
2.  [Model Architecture](#model-architecture)
3.  [Evaluation Metrics](#evaluation-metrics)
4.  [Usage](#usage)
5.  [Setup Instructions (Implicit)](#setup-instructions-implicit)

## Project Overview

The goal of this project is to create an effective secret detection system. It leverages a Convolutional Neural Network (CNN) trained on a custom dataset (`Podric/prowl-secrets-corpus`) to identify sensitive information in text. The process involves data loading, preprocessing, model training, evaluation, and optimizing a classification threshold.

## Model Architecture

The core of the secret detection system is a 1D Convolutional Neural Network (CNN).

*   **Text Vectorization**: Converts raw text into numerical sequences using a vocabulary of up to 50,000 tokens and a maximum sequence length of 300.
*   **Embedding Layer**: Maps token IDs to a dense vector representation of 128 dimensions.
*   **Convolutional Layers**: Two 1D convolutional layers with `filters=128` and `kernel_sizes=5` and `3` respectively, followed by Batch Normalization and MaxPooling to extract hierarchical features from the text.
*   **Global MaxPooling**: Aggregates features from the convolutional layers.
*   **Dense Layers**: Fully connected layers with ReLU activation for classification.
*   **Output Layer**: A single neuron with a sigmoid activation function outputs the probability of a secret being present.

## Evaluation Metrics

The model's performance was evaluated using the following metrics, achieving the indicated values on the test set:

*   **Accuracy**: 0.8828
*   **Precision**: 0.8610
*   **Recall**: 0.8053
*   **F1-score**: 0.8335
*   **ROC-AUC**: 0.9547
*   **PR-AUC**: 0.9323
*   **Best Threshold**: 0.5400

The `Best Threshold` of 0.5400 was determined by maximizing the F1-score on the validation set, offering a balance between precision and recall.

## Usage

To use the trained model for secret detection, you can utilize the `detect_secret` function:

```python
# Load the model and configuration (as done in the notebook)
# model = tf.keras.models.load_model('/content/secret_detector_cnn.keras')
# BEST_THRESHOLD = config['threshold'] # from loaded model_config.json

def detect_secret(text, threshold=BEST_THRESHOLD):
    """
    Detects if a secret is present in the given text.

    Args:
        text (str): The input text to scan.
        threshold (float): The classification threshold to use. Defaults to BEST_THRESHOLD.

    Returns:
        dict: A dictionary containing the prediction ('SECRET' or 'SAFE'),
              confidence, secret probability, and the threshold used.
    """
    probability = float(
        model.predict(
            tf.constant([str(text)], dtype=tf.string)
        )[0][0]
    )

    is_secret = (
        probability >= threshold
    )

    if is_secret:
        prediction = "SECRET"
        confidence = probability
    else:
        prediction = "SAFE"
        confidence = 1 - probability

    return {
        "prediction": prediction,
        "confidence": round(confidence * 100, 2),
        "secret_probability": round(probability * 100, 2),
        "threshold": round(threshold, 3)
    }

# Example usage:
example_text_secret = "My API key is sk_live_xxxxxxxxxxxxxxxxxxxx"
result_secret = detect_secret(example_text_secret)
print(f"Secret example: {result_secret}")

example_text_safe = "This is a safe sentence."
result_safe = detect_secret(example_text_safe)
print(f"Safe example: {result_safe}")

# To scan multiple lines:
def scan_text(text, threshold=BEST_THRESHOLD):
    lines = str(text).splitlines()
    findings = []
    for line_number, line in enumerate(lines, start=1):
        if not line.strip():
            continue
        result = detect_secret(line, threshold=threshold)
        if result["prediction"] == "SECRET":
            findings.append({
                "line": line_number,
                "text": line,
                "confidence": result["confidence"]
            })
    return findings

sample_code = """
def calculate(a, b):
    return a + b

API_KEY = "example-test-value"

def hello():
    print("Hello")

DATABASE_PASSWORD = "example-password"
"""

scan_results = scan_text(sample_code)
print(f"\nScan results for sample_code: {scan_results}")
```

## Setup Instructions (Implicit)

This notebook already contains all the necessary code to load data, preprocess it, define and train the model, and evaluate its performance. To replicate the environment and run the code, ensure you have:

*   Google Colab environment (or a similar Python environment).
*   Required libraries installed (e.g., `tensorflow`, `pandas`, `sklearn`, `datasets`, `matplotlib`, `seaborn`). These are typically installed via `pip install <library_name>`.

All data loading, model definition, training, and evaluation steps are executed sequentially within the notebook.

