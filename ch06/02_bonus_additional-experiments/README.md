# Additional Experiments

The table below adds experiments to answer additional questions about various design choices. The first row uses the same settings as the main chapter and is used as a reference.
For example, 

- comparing rows 1 and 2 answers the question: "What is the performance difference when we train the last or first token?";
- comparing rows 1 and 3 answers the question: "What is the performance difference when we train only the last layer instead of the last block?";
- and so forth.

&nbsp;

|    | Model              | Weights    | Trainable token | Trainable layers | Context length          | Training acc | Test acc | Validation acc | Training time | CPU/GPU |
|----|--------------------|------------|-----------------|------------------|-------------------------|--------------|----------|----------------|---------------|---------|
| 1  | gpt2-small (124M)  | pretrained | last            | last_block       | longest train ex. (120) | 96.63%       | 95.00%   | 99.33%         | 0.39 min      | V100    |
| 2  | gpt2-small (124M)  | pretrained | first           | last_block       | longest train ex. (120) | 78.46%       | 75.00%   | 80.54%         | 0.37 min      | V100    |
| 3  | gpt2-small (124M)  | pretrained | last            | last_layer       | longest train ex. (120) | 78.65%       | 72.00%   | 79.87%         | 0.33 min      | V100    |
| 4  | gpt2-small (124M)  | pretrained | last            | all              | longest train ex. (120) | 99.62%       | 96.67%   | 96.64%         | 0.94 min      | V100    |
| 5  | gpt2-medium (355M) | pretrained | last            | last_block       | longest train ex. (120) | 87.50%       | 84.67%   | 91.28%         | 0.91 min      | V100    |
| 6  | gpt2-large (774M)  | pretrained | last            | last_block       | longest train ex. (120) | 99.52%       | 96.67%   | 98.66%         | 1.91 min      | V100    |
| 7  | gpt2-xl (1558M)    | pretrained | last            | last_block       | longest train ex. (120) | 99.81%       | 98.33%   | 99.33%         | 3.84 min      | V100    |
| 8  | gpt2-small (124M)  | random     | last            | all              | longest train ex. (120) | 100%         | 93.67%   | 96.64%         | 0.93 min      | V100    |
| 9  | gpt2-small (124M)  | pretrained | last            | LoRA             | longest train ex. (120) | 99.52%       | 97.67%   | 97.99%         | 0.82 min      | V100    |
| 10 | gpt2-small (124M)  | pretrained | last            | last_block       | context length (1024)   | 83.08%       | 78.33%   | 87.92%         | 3.24 min      | V100    |


&nbsp;

### Usage

You can use the following code to reproduce the experiments:

- Row 1: `python additional-experiments.py`
- Row 2: `python additional-experiments.py --trainable_token first` 
- Row 3: `python additional-experiments.py --trainable_layers last_layer`
- Row 4: `python additional-experiments.py --trainable_layers all`
- Row 5: `python additional-experiments.py --model_size "gpt2-medium (355M)"`
- Row 6: `python additional-experiments.py --model_size "gpt2-large (774M)"`
- Row 7: `python additional-experiments.py --model_size "gpt2-xl (1558M)"`
- Row 8: `python additional-experiments.py --weights random --trainable_layers all`
- Row 9: `python additional-experiments.py --trainable_layers lora --lora_rank 16 --lora_alpha 8`
- Row 10: `python additional-experiments.py --context_length "model_context_length"`

I've kept the LLM and dataset small on purpose, so you can run the training on a regular laptop like a MacBook Air M3 in about 15 minutes in case you don't have access to a GPU.

&nbsp;

### Interpretation

1. **Training the Last vs. First Output Token (Row 1 vs. 2)**: Training the last output token results in substantially better performance compared to the first. This improvement is expected due to the causal self-attention mask.

2. **Training the Last Transformer Block vs. Last Layer (Row 1 vs. 3)**: Training the entire last transformer block is also results in substantially better results than training only the last layer.

3. **Training All Layers vs. Last Transformer Block (Row 1 vs. 4)**: Training all layers shows a modest improvement of ~2% over just training the last transformer block, but it requires almost three times longer in terms of training duration.

4. **Using Larger Pretrained Models (Row 1 vs 5, and Row 1 vs. 6 and 7)**: Employing a 3x larger pretrained model leads to worse results. However, using a 5x larger model improves performance compared to the initial model, as was anticipated. Similarly, the 12x larger model improves the predictive performance even further. (The medium model was perhaps not well pretrained or the particular finetuning configuration works not as well for this model.)

5. **Using a Model with Random Weights vs. Pretrained Weights (Row 1 vs. 8)**: Utilizing a model with random weights yields results that are only slightly worse by 1.3% compared to using pretrained weights.

6. **Using LoRA (Low-Rank Adaptation) vs Training All Layers (Row 9 vs. 4)**: Keeping the model frozen and adding trainable LoRA layers (see [Appendix E](../../appendix-E/01_main-chapter-code/appendix-E.ipynb) for details) is a viable alternative to training all model parameters and even improves the performance by 1% point. As it can be seen by the 1% lower gap between the training and validation accuracy when using LoRA, this is likely due to less overfitting. Moreover, using LoRA is also slightly faster because fewer parameters have to be updated.

7. **Padding Input to Full Context Length vs. Longest Training Example (Row 1 vs. 10)**: Padding the input to the full supported context length results is significantly worse.
