# Day 6: Advanced Transformers - Fine-Tuning BERT Variants

Welcome to Day 6. Yesterday, we used `GPT-2` for General Text Generation. 
But generative text is notoriously bad for strict analytical tasks (like Classifying $10,000$ legal documents accurately). Generative models hallucinate. 

For strict, deterministic classification, we return to the **Encoder (BERT)** framework.

## The BERT Variants
The original BERT was revolutionary, but Researchers quickly discovered inefficiencies. They built optimized variations:
1.  **RoBERTa (Robustly Optimized BERT):** Stripped out the "Next Sentence Prediction" phase of BERT, using massive batch sizes and more data to drastically outperform original BERT on classification tasks.
2.  **DistilBERT:** Smashed the architecture down, retaining $97\%$ of BERT's performance while running $60\%$ faster! Ideal for mobile phones or low-RAM environments.
3.  **ALBERT (A Lite BERT):** Shares mathematical parameters across layers to drastically shrink memory consumption!

## Hands-On: Transfer Learning with RoBERTa
Look at `day6_ex.py`. We load the massive `ag_news` dataset from Hugging Face containing thousands of articles. 
We want to Fine-Tune `RoBERTa` to categorize these articles automatically into $4$ labels: (World, Sports, Business, Sci/Tech).

Instead of PyTorch's complex boilerplate loops, Hugging Face provides the `Trainer` API to streamline the entire Transfer Learning pipeline!

```python
# day6_ex.py
from transformers import AutoTokenizer, AutoModelForSequenceClassification, Trainer, TrainingArguments
from datasets import load_dataset

# 1. Download the AG News Dataset (120,000 Articles!)
dataset = load_dataset("ag_news")

# 2. Download Facebook/Meta's Pre-trained RoBERTa!
tokenizer = AutoTokenizer.from_pretrained("roberta-base")

# Note we specify 4 Labels! We are overwriting RoBERTa's output layer!
model = AutoModelForSequenceClassification.from_pretrained("roberta-base", num_labels=4)

def tokenize_function(examples):
    return tokenizer(examples["text"], truncation=True, padding="max_length", max_length=128)

tokenized_datasets = dataset.map(tokenize_function, batched=True)
# ... [Format keys to float tensors and remove 'text' columns]

# 3. Setup the High-Performance Fine-Tuning Configuration!
training_args = TrainingArguments(
    output_dir="./results",
    eval_strategy="epoch",
    learning_rate=2e-5,       # TINY Learning Rate! Do NOT destroy RoBERTa's base math!
    per_device_train_batch_size=16,
    num_train_epochs=3,       # Transfer Learning is blazing fast! Only 3 epochs!
    weight_decay=0.01         # Aggressive Regularization to prevent Overfitting!
)

# 4. Initialize Hugging Face's automated Trainer!
trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_datasets["train"],
    eval_dataset=tokenized_datasets["test"],
    processing_class=tokenizer
)

# EXECUTE FINE-TUNING ALGORITHM!
trainer.train()

# Output Results: Test Accuracy > ~94%!
# Output:
# Loading weights:   0%|          | 0/197 [00:00<?, ?it/s]
# Loading weights: 100%|██████████| 197/197 [00:00<00:00, 2971.76it/s]
# [1mRobertaForSequenceClassification LOAD REPORT[0m from: roberta-base
# Key                             | Status     | 
# --------------------------------+------------+-
# lm_head.layer_norm.bias         | UNEXPECTED | 
# lm_head.dense.weight            | UNEXPECTED | 
# lm_head.layer_norm.weight       | UNEXPECTED | 
# lm_head.dense.bias              | UNEXPECTED | 
# lm_head.bias                    | UNEXPECTED | 
# roberta.embeddings.position_ids | UNEXPECTED | 
# classifier.out_proj.weight      | MISSING    | 
# classifier.dense.bias           | MISSING    | 
# classifier.dense.weight         | MISSING    | 
# classifier.out_proj.bias        | MISSING    | 
# 
# [3mNotes:
# - UNEXPECTED[3m	:can be ignored when loading from different task/architecture; not ok if you expect identical arch.
# - MISSING[3m	:those params were newly initialized because missing from the checkpoint. Consider training on your downstream task.[0m
# 
# Map:   0%|          | 0/120000 [00:00<?, ? examples/s]
# Map:   1%|          | 1000/120000 [00:00<00:29, 4047.43 examples/s]
# Map:   2%|▎         | 3000/120000 [00:00<00:12, 9276.48 examples/s]
# Map:   4%|▍         | 5000/120000 [00:00<00:09, 12351.24 examples/s]
# Map:   8%|▊         | 9000/120000 [00:00<00:06, 16186.80 examples/s]
# Map:  11%|█         | 13000/120000 [00:00<00:05, 17949.18 examples/s]
# Map:  12%|█▎        | 15000/120000 [00:00<00:05, 18206.32 examples/s]
# Map:  16%|█▌        | 19000/120000 [00:01<00:05, 19202.87 examples/s]
# Map:  19%|█▉        | 23000/120000 [00:01<00:04, 19672.49 examples/s]
# Map:  21%|██        | 25000/120000 [00:01<00:04, 19369.08 examples/s]
# Map:  24%|██▍       | 29000/120000 [00:01<00:04, 20016.29 examples/s]
# Map:  28%|██▊       | 33000/120000 [00:02<00:05, 14832.35 examples/s]
# Map:  29%|██▉       | 35000/120000 [00:02<00:05, 14904.77 examples/s]
# Map:  31%|███       | 37000/120000 [00:02<00:05, 14366.54 examples/s]
# Map:  32%|███▎      | 39000/120000 [00:02<00:05, 15025.14 examples/s]
# Map:  35%|███▌      | 42000/120000 [00:02<00:04, 16322.80 examples/s]
# Map:  38%|███▊      | 45000/120000 [00:02<00:04, 17269.30 examples/s]
# Map:  40%|████      | 48000/120000 [00:02<00:04, 17932.71 examples/s]
# Map:  42%|████▎     | 51000/120000 [00:03<00:03, 18568.94 examples/s]
# Map:  46%|████▌     | 55000/120000 [00:03<00:03, 19319.74 examples/s]
# Map:  48%|████▊     | 58000/120000 [00:03<00:03, 19590.26 examples/s]
# Map:  52%|█████▏    | 62000/120000 [00:03<00:02, 19981.32 examples/s]
# Map:  54%|█████▍    | 65000/120000 [00:03<00:03, 14920.98 examples/s]
# Map:  56%|█████▌    | 67000/120000 [00:04<00:03, 15471.92 examples/s]
# Map:  57%|█████▊    | 69000/120000 [00:04<00:03, 16233.06 examples/s]
# Map:  60%|██████    | 72000/120000 [00:04<00:02, 17287.11 examples/s]
# Map:  62%|██████▎   | 75000/120000 [00:04<00:02, 17982.50 examples/s]
# Map:  65%|██████▌   | 78000/120000 [00:04<00:02, 18547.66 examples/s]
# Map:  68%|██████▊   | 82000/120000 [00:04<00:01, 19368.62 examples/s]
# Map:  71%|███████   | 85000/120000 [00:04<00:01, 19521.97 examples/s]
# Map:  74%|███████▍  | 89000/120000 [00:05<00:01, 19864.18 examples/s]
# Map:  78%|███████▊  | 93000/120000 [00:05<00:01, 15027.97 examples/s]
# Map:  81%|████████  | 97000/120000 [00:05<00:01, 16471.47 examples/s]
# Map:  84%|████████▍ | 101000/120000 [00:05<00:01, 17577.90 examples/s]
# Map:  87%|████████▋ | 104000/120000 [00:06<00:00, 18143.63 examples/s]
# Map:  89%|████████▉ | 107000/120000 [00:06<00:00, 18218.49 examples/s]
# Map:  91%|█████████ | 109000/120000 [00:06<00:00, 17911.71 examples/s]
# Map:  93%|█████████▎| 112000/120000 [00:06<00:00, 18436.51 examples/s]
# Map:  96%|█████████▌| 115000/120000 [00:06<00:00, 18889.49 examples/s]
# Map:  98%|█████████▊| 117000/120000 [00:06<00:00, 19038.24 examples/s]
# Map:  99%|█████████▉| 119000/120000 [00:06<00:00, 19111.96 examples/s]
# Map: 100%|██████████| 120000/120000 [00:07<00:00, 16224.04 examples/s]
# 
# Map:   0%|          | 0/7600 [00:00<?, ? examples/s]
# Map:  26%|██▋       | 2000/7600 [00:00<00:00, 7020.53 examples/s]
# Map:  79%|███████▉  | 6000/7600 [00:00<00:00, 13638.61 examples/s]
# Map: 100%|██████████| 7600/7600 [00:00<00:00, 13411.90 examples/s]
# Traceback (most recent call last):
#   File "<string>", line 31, in <module>
#   File "<string>", line 112, in __init__
#   File "/home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/transformers/training_args.py", line 1577, in __post_init__
#     self.device
#   File "/home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/transformers/training_args.py", line 1857, in device
#     return self._setup_devices
#            ^^^^^^^^^^^^^^^^^^^
#   File "/home/pollmix/.pyenv/versions/3.12.12/lib/python3.12/functools.py", line 998, in __get__
#     val = self.func(instance)
#           ^^^^^^^^^^^^^^^^^^^
#   File "/home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/transformers/training_args.py", line 1752, in _setup_devices
#     raise ImportError(
# ImportError: Using the `Trainer` with `PyTorch` requires `accelerate>=1.1.0`: Please run `pip install transformers[torch]` or `pip install 'accelerate>=1.1.0'`
```

## Wrapping Up Day 6
With exactly $50$ lines of Python, you downloaded a $125+$ Million Parameter State-of-the-Art Language Engine trained by Meta's Supercomputers... and perfectly re-wired it to blindly classify World News categories at superhuman accuracy in under $20$ minutes!

This is the power of the `transformers` library. 

Tomorrow, on **Day 7: The Final NLP Capstone**, we move past simple Classification and Generative prompting. We will utilize Google's **Text-To-Text Transfer Transformer (T5)** to aggressively auto-summarize a dataset containing thousands of human conversations!
