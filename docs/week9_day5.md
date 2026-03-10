# Day 5: Building Neural Networks in TensorFlow

Welcome to Day 5. The theoretical mathematics is behind us. Today, we harness the power of **TensorFlow** (Google) to build a sophisticated architecture natively. 

TensorFlow handles all the Forward Propagation, Backpropagation, and Adam Optimization completely transparently behind the scenes.

## The Keras API
Inside TensorFlow is an API called **Keras** (`tf.keras`). It allows you to build a Neural Network simply by stacking layers like Lego blocks!

Let's look at `day5_ex.py`. Our goal is to classify handwritten digits out of the `MNIST` dataset using a `Sequential` architecture.

```python
# day5_ex.py
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Dense, Flatten, Conv2D, MaxPooling2D, Dropout

# 1. Initialize the Keras Architecture
model = Sequential([
    # Input Layer (Flattens 28x28 pixel images into 784 raw neurons)
    Flatten(input_shape=(28, 28, 1)),
    
    # Hidden Layer (128 Neurons using ReLU Non-Linearity)
    Dense(128, activation="relu"),
    
    # Regularization Layer (Mathematically "kills" 50% of 
    # the active neurons randomly to prevent Overfitting!)
    Dropout(0.5),
    
    # Output Layer (10 Neurons using Softmax percent conversion!)
    Dense(10, activation="softmax")   
])

# 2. Check the blueprint
model.summary()

# 3. Assemble the mathematical logic (The "Compile" phase)
model.compile(
    optimizer="adam",                  # The Math Fixer
    loss="categorical_crossentropy",   # The Error Metric
    metrics=['accuracy']
)

# 4. TRAIN THE SUPERCONDUCTOR!
history = model.fit(
    X_train, y_train,
    epochs=10,        # Rerun Forward/Backward Prop 10 times over the whole dataset!
    batch_size=32,    # Only process 32 images at a time (SGD rule)
    validation_split=0.2
)

# Output: Epoch 10/10 - val_accuracy: 0.9850 (98.5% Accuracy!)
# Output:
# Model: "sequential"
# ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━┓
# ┃ Layer (type)                    ┃ Output Shape           ┃       Param # ┃
# ┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━┩
# │ flatten (Flatten)               │ (None, 784)            │             0 │
# ├─────────────────────────────────┼────────────────────────┼───────────────┤
# │ dense (Dense)                   │ (None, 128)            │       100,480 │
# ├─────────────────────────────────┼────────────────────────┼───────────────┤
# │ dropout (Dropout)               │ (None, 128)            │             0 │
# ├─────────────────────────────────┼────────────────────────┼───────────────┤
# │ dense_1 (Dense)                 │ (None, 10)             │         1,290 │
# └─────────────────────────────────┴────────────────────────┴───────────────┘
#  Total params: 101,770 (397.54 KB)
#  Trainable params: 101,770 (397.54 KB)
#  Non-trainable params: 0 (0.00 B)
# 2026-03-09 23:57:10.892406: I tensorflow/core/platform/cpu_feature_guard.cc:210] This TensorFlow binary is optimized to use available CPU instructions in performance-critical operations.
# To enable the following instructions: AVX2 FMA, in other operations, rebuild TensorFlow with the appropriate compiler flags.
# /home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/keras/src/layers/reshaping/flatten.py:37: UserWarning: Do not pass an `input_shape`/`input_dim` argument to a layer. When using Sequential models, prefer using an `Input(shape)` object as the first layer in the model instead.
#   super().__init__(**kwargs)
# WARNING: All log messages before absl::InitializeLog() is called are written to STDERR
# I0000 00:00:1773115033.963980  745187 gpu_device.cc:2020] Created device /job:localhost/replica:0/task:0/device:GPU:0 with 6085 MB memory:  -> device: 0, name: NVIDIA GeForce RTX 2080 SUPER, pci bus id: 0000:05:00.0, compute capability: 7.5
# 2026-03-09 23:57:14.263011: W external/local_xla/xla/service/gpu/llvm_gpu_backend/default/nvptx_libdevice_path.cc:41] Can't find libdevice directory ${CUDA_DIR}/nvvm/libdevice. This may result in compilation or runtime failures, if the program we try to run uses routines from libdevice.
# Searched for CUDA in the following directories:
#   ./cuda_sdk_lib
#   
# import os
# import matplotlib
# matplotlib.use('Agg')
# import matplotlib.pyplot as plt
# def _custom_show(*args, **kwargs):
#     plt.savefig('/home/pollmix/Coding/AI-Engineer-Bootcamp/docs/images/plot_7327cc42.png')
#     print('__PLOT_SAVED__')
# plt.show = _custom_show
# 
# # day5_ex.py
# from tensorflow.keras.models import Sequential
# from tensorflow.keras.layers import Dense, Flatten, Conv2D, MaxPooling2D, Dropout
# 
# # 1. Initialize the Keras Architecture
# model = Sequential([
#     # Input Layer (Flattens 28x28 pixel images into 784 raw neurons)
#     Flatten(input_shape=(28, 28, 1)),
#     
#     # Hidden Layer (128 Neurons using ReLU Non-Linearity)
#     Dense(128, activation="relu"),
#     
#     # Regularization Layer (Mathematically "kills" 50% of 
#     # the active neurons randomly to prevent Overfitting!)
#     Dropout(0.5),
#     
#     # Output Layer (10 Neurons using Softmax percent conversion!)
#     Dense(10, activation="softmax")   
# ])
# 
# # 2. Check the blueprint
# model.summary()
# 
# # 3. Assemble the mathematical logic (The "Compile" phase)
# model.compile(
#     optimizer="adam",                  # The Math Fixer
#     loss="categorical_crossentropy",   # The Error Metric
#     metrics=['accuracy']
# )
# 
# # 4. TRAIN THE SUPERCONDUCTOR!
# history = model.fit(
#     X_train, y_train,
#     epochs=10,        # Rerun Forward/Backward Prop 10 times over the whole dataset!
#     batch_size=32,    # Only process 32 images at a time (SGD rule)
#     validation_split=0.2
# )
# 
# # Output: Epoch 10/10 - val_accuracy: 0.9850 (98.5% Accuracy!).runfiles/cuda_nvcc
#   
# import os
# import matplotlib
# matplotlib.use('Agg')
# import matplotlib.pyplot as plt
# def _custom_show(*args, **kwargs):
#     plt.savefig('/home/pollmix/Coding/AI-Engineer-Bootcamp/docs/images/plot_7327cc42.png')
#     print('__PLOT_SAVED__')
# plt.show = _custom_show
# 
# # day5_ex.py
# from tensorflow.keras.models import Sequential
# from tensorflow.keras.layers import Dense, Flatten, Conv2D, MaxPooling2D, Dropout
# 
# # 1. Initialize the Keras Architecture
# model = Sequential([
#     # Input Layer (Flattens 28x28 pixel images into 784 raw neurons)
#     Flatten(input_shape=(28, 28, 1)),
#     
#     # Hidden Layer (128 Neurons using ReLU Non-Linearity)
#     Dense(128, activation="relu"),
#     
#     # Regularization Layer (Mathematically "kills" 50% of 
#     # the active neurons randomly to prevent Overfitting!)
#     Dropout(0.5),
#     
#     # Output Layer (10 Neurons using Softmax percent conversion!)
#     Dense(10, activation="softmax")   
# ])
# 
# # 2. Check the blueprint
# model.summary()
# 
# # 3. Assemble the mathematical logic (The "Compile" phase)
# model.compile(
#     optimizer="adam",                  # The Math Fixer
#     loss="categorical_crossentropy",   # The Error Metric
#     metrics=['accuracy']
# )
# 
# # 4. TRAIN THE SUPERCONDUCTOR!
# history = model.fit(
#     X_train, y_train,
#     epochs=10,        # Rerun Forward/Backward Prop 10 times over the whole dataset!
#     batch_size=32,    # Only process 32 images at a time (SGD rule)
#     validation_split=0.2
# )
# 
# # Output: Epoch 10/10 - val_accuracy: 0.9850 (98.5% Accuracy!).runfiles/cuda_nvdisasm
#   
# import os
# import matplotlib
# matplotlib.use('Agg')
# import matplotlib.pyplot as plt
# def _custom_show(*args, **kwargs):
#     plt.savefig('/home/pollmix/Coding/AI-Engineer-Bootcamp/docs/images/plot_7327cc42.png')
#     print('__PLOT_SAVED__')
# plt.show = _custom_show
# 
# # day5_ex.py
# from tensorflow.keras.models import Sequential
# from tensorflow.keras.layers import Dense, Flatten, Conv2D, MaxPooling2D, Dropout
# 
# # 1. Initialize the Keras Architecture
# model = Sequential([
#     # Input Layer (Flattens 28x28 pixel images into 784 raw neurons)
#     Flatten(input_shape=(28, 28, 1)),
#     
#     # Hidden Layer (128 Neurons using ReLU Non-Linearity)
#     Dense(128, activation="relu"),
#     
#     # Regularization Layer (Mathematically "kills" 50% of 
#     # the active neurons randomly to prevent Overfitting!)
#     Dropout(0.5),
#     
#     # Output Layer (10 Neurons using Softmax percent conversion!)
#     Dense(10, activation="softmax")   
# ])
# 
# # 2. Check the blueprint
# model.summary()
# 
# # 3. Assemble the mathematical logic (The "Compile" phase)
# model.compile(
#     optimizer="adam",                  # The Math Fixer
#     loss="categorical_crossentropy",   # The Error Metric
#     metrics=['accuracy']
# )
# 
# # 4. TRAIN THE SUPERCONDUCTOR!
# history = model.fit(
#     X_train, y_train,
#     epochs=10,        # Rerun Forward/Backward Prop 10 times over the whole dataset!
#     batch_size=32,    # Only process 32 images at a time (SGD rule)
#     validation_split=0.2
# )
# 
# # Output: Epoch 10/10 - val_accuracy: 0.9850 (98.5% Accuracy!).runfiles/nvidia_nvshmem
#   
# import/cuda_nvcc
#   
# import/cuda_nvdisasm
#   
# import/nvidia_nvshmem
#   
#   /usr/local/cuda
#   /opt/cuda
#   /home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/tensorflow/python/platform/../../../nvidia/cuda_nvcc
#   /home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/tensorflow/python/platform/../../../../nvidia/cuda_nvcc
#   /home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/tensorflow/python/platform/../../cuda
#   /home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/tensorflow/python/platform/../../../../../..
#   /home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/tensorflow/python/platform/../../../../../../..
#   .
# You can choose the search directory by setting xla_gpu_cuda_data_dir in HloModule's DebugOptions.  For most apps, setting the environment variable XLA_FLAGS=--xla_gpu_cuda_data_dir=/path/to/cuda will work.
# Traceback (most recent call last):
#   File "<string>", line 43, in <module>
# NameError: name 'X_train' is not defined
```

## Wrapping Up Day 5
With a shocking 20 lines of code, you have built a model identifying thousands of handwritten numbers with $98\%$ accuracy! 

Because Keras handled the Calculus backpropagation, your main job is simply engineering the Architecture. You experiment by changing `128` Neurons to `256` Neurons. You experiment by modifying `Dropout(0.5)`. 

TensorFlow natively dominates enterprise AI. But there is a massive competitor built by Facebook that has completely monopolized the Research sector. 

Tomorrow, on **Day 6: PyTorch**, we will rebuild the exact same Neural Network natively inside PyTorch!
