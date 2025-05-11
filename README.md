# Hierarchical CTC OCR Model

This project implements an OCR model for Vietnamese text using a Hierarchical CTC approach. The goal is to improve character and diacritic recognition by structuring the prediction heads hierarchically while still using a standard CTC loss on a combined character vocabulary.

## Architecture

*   **Vision Encoder:** Extracts frame-level visual features.
*   **Intermediate RNN:** Processes the visual sequence.
*   **Shared Feature Layer:** A common layer processing RNN output before branching.
*   **Hierarchical Heads:**
    *   **Base Head:** Predicts base character probabilities from shared features.
    *   **Conditioning:** Uses shared features and potentially base predictions to create conditioned features for the diacritic head (e.g., via concatenation or gating).
    *   **Diacritic Head:** Predicts diacritic probabilities from conditioned features.
*   **Final Classifier Head:** Takes features derived from the hierarchical structure (e.g., the shared features, or a combination of features/logits from branches) and predicts probabilities over a **combined character vocabulary** (e.g., 'a', 'á', 'à', 'â', 'b', ...).
*   **CTC Loss:** Trained using standard CTC loss on the output of the final combined classifier head against ground truth labels composed of combined character indices.

## Purpose

This architecture attempts to leverage the linguistic structure (Diacritic depends on Base) *internally* by conditioning the diacritic prediction path on the base prediction path (or related features). By training end-to-end with a CTC loss on the final combined characters, it avoids the composition and alignment issues of the Dual CTC approach during inference, while still encouraging the internal layers to learn the base-diacritic relationship.

## Key Differences from Dual CTC

*   **Single Final Output/Loss:** Uses one final classifier head predicting combined characters and a single CTC loss, simplifying training and inference decoding.
*   **Internal Hierarchy:** Explicitly structures the layers *before* the final classifier to model the base-diacritic dependency (e.g., via conditioning).
*   **No Post-Composition:** Inference uses standard CTC decoding on the combined character output, eliminating the need for a separate composition step.

## Project Structure

(Similar structure to `ctc_ocr` and `dual_ctc_ocr`, with model/scripts renamed)

```bash
./hierarchical_ctc_ocr/
├── model/
│   ├── __init__.py
│   └── hierarchical_ctc_model.py # Hierarchical model definition
├── data/
│   ├── __init__.py
│   ├── ctc_ocr_dataset.py      # Reused: Expects combined char labels
│   └── ctc_collation.py        # Reused: Standard CTC collation
├── training/
│   ├── __init__.py
│   ├── ctc_trainer.py          # Reused: Standard CTC trainer
│   └── ctc_validation.py       # Reused: Standard CTC validation
├── utils/
│   ├── __init__.py
│   ├── optimizers.py           # Reused
│   ├── schedulers.py           # Reused
│   └── ctc_utils.py            # Reused (needs combined vocab builder)
├── scripts/
│   ├── __init__.py
│   ├── train_hierarchical_ctc.py # Main training script
│   └── evaluate_hierarchical_ctc.py# Evaluation script
├── outputs/
│   └── hierarchical_ctc_ocr_model/
├── logs/
│   └── main_training_hierarchical.log
├── README_hierarchical.md      # This file
└── requirements_hierarchical.txt # Dependencies



```bash
python -m scripts.train_hierarchical_ctc   --dataset_name vklinhhh/vnhwt_opt_1    --output_dir outputs/dynamic_fusion_large  --use_dynamic_fusion   --use_feature_enhancer   --transformer_d_model 768   --transformer_nhead 12   --num_transformer_layers 6   --shared_hidden_size 768   --batch_size 16   --learning_rate 5e-5   --discriminative_lr   --encoder_lr_factor 0.1   --warmup_ratio 0.05   --weight_decay 0.02   --epochs 25   --use_visual_diacritic_attention   --use_character_diacritic_compatibility   --use_few_shot_diacritic_adapter   --num_few_shot_prototypes 10   --use_amp   --fusion_method concat_proj

