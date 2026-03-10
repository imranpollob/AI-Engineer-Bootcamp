# Day 1: Introduction to Sequence Modeling and RNNs

Welcome to Week 11. Until today, our Neural Networks evaluated data independently. The 10th photograph of a dog didn't rely on the 9th photograph. 

But in **Sequence Modeling**, the current data point is completely dependent on the data point that came before it. If I say the word "United", there is a high mathematical probability the next word is "States".

How do we grant an AI the ability to remember the word "United"? We use a **Recurrent Neural Network (RNN)**.

## The Recurrent Mechanism
A standard Dense layer takes Input $X$, multiplies it by Weights $W$, and produces Output $y$. It forgets $X$ immediately.

A **Recurrent Layer** takes Input $X_{1}$ and produces Output $y_{1}$. BUT, it also saves a copy of its mathematical state (called a **Hidden State**). 
When Input $X_{2}$ arrives, the RNN does not evaluate $X_{2}$ blindly! It combines $X_{2}$ with the Hidden State of $X_{1}$! 

The prediction $y_{2}$ is mathematically influenced by the memory of $X_{1}$!

## Hands-On: IMDB Movie Reviews
Look at `day1_tensorflow.py` and `day1_pytorch.py`. We load the famous IMDB dataset.

```python
# dat1_tensorflow.py
from tensorflow.keras.datasets import imdb
from tensorflow.keras.preprocessing.sequence import pad_sequences
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Embedding, SimpleRNN, Dense

# 1. Load the top 10,000 most common English words
vocab_size = 10000
(X_train, y_train), (X_test, y_test) = imdb.load_data(num_words=vocab_size)

# 2. Pad the Sequences! (Every movie review must be exactly 200 words long)
max_len = 200
X_train = pad_sequences(X_train, maxlen=max_len, padding="post")

# 3. Build the RNN Pipeline
model = Sequential([
    # Turn English words into mathematical vectors!
    Embedding(input_dim=vocab_size, output_dim=128),
    
    # THE RECURRENT MEMORY LAYER!
    SimpleRNN(128, activation='tanh', return_sequences=False),
    
    # Is the review Positive (1) or Negative (0)?
    Dense(1, activation='sigmoid')
])
# Output:
# 2026-03-09 23:56:44.692658: I tensorflow/core/platform/cpu_feature_guard.cc:210] This TensorFlow binary is optimized to use available CPU instructions in performance-critical operations.
# To enable the following instructions: AVX2 FMA, in other operations, rebuild TensorFlow with the appropriate compiler flags.
# /home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/numpy/lib/_format_impl.py:838: VisibleDeprecationWarning: dtype(): align should be passed as Python or NumPy boolean but got `align=0`. Did you mean to pass a tuple to create a subarray type? (Deprecated NumPy 2.4)
#   array = pickle.load(fp, **pickle_kwargs)
# WARNING: All log messages before absl::InitializeLog() is called are written to STDERR
# I0000 00:00:1773115010.875610  744723 gpu_device.cc:2020] Created device /job:localhost/replica:0/task:0/device:GPU:0 with 6086 MB memory:  -> device: 0, name: NVIDIA GeForce RTX 2080 SUPER, pci bus id: 0000:05:00.0, compute capability: 7.5
```

### What is `pad_sequences`?
A Neural Network requires fixed-size Matrix arrays. But humans write reviews of varying lengths! One review is 15 words; another is 500 words. 
`pad_sequences(maxlen=200)` forces every review to be exactly 200 words long. If it's shorter, it pads the end with invisible `0`s. If it's longer, it brutally truncates it.

## Wrapping Up Day 1
You have built your first "Memory" network utilizing TensorFlow's `SimpleRNN` and PyTorch's `nn.RNN`. It achieved roughly 80% accuracy guessing if a movie review was good or bad!

But how does it train this memory? Backpropagation worked great for static images, but how do we calculate Calculus across the passage of Time?

Tomorrow on **Day 2: Architecture and BPTT**, we explore the mathematical limitations of `SimpleRNN` and why it suffers from "Amnesia".
