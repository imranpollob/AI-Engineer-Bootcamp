# Day 4: Building CNN Architectures with Keras and TensorFlow

Welcome to Day 4. Today we build a completely functional Convolutional Neural Network (CNN) using `TensorFlow` and `Keras`!

We return to our `CIFAR-10` dataset containing 60,000 $32 \times 32$ pixel images of 10 different objects (Airplanes, Dogs, Frogs, etc.).

## The Architecture
Let's assemble the $3$-part pipeline discussed on Day 1 using `day4_ex.py`. 

```python
# day4_ex.py
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import Conv2D, MaxPooling2D, Flatten, Dense, Dropout

# 1. Initialize the Keras Architecture
model = Sequential([
    # PART 1: The First Feature Extraction Block
    # 32 Different sliding windows (kernels) processing 3-Color Channel input!
    Conv2D(32, (3, 3), activation='relu', input_shape=(32,32,3)),
    # Compress the (32x32) image down into a (16x16) array of brightest features!
    MaxPooling2D((2, 2)),
    
    # PART 2: The Deep Feature Extraction Block
    # Look deeper! Now use 64 different sliding windows on the compressed data!
    Conv2D(64, (3, 3), activation='relu'),
    # Compress the (16x16) data down into a tiny (8x8) array!
    MaxPooling2D((2, 2)),
    
    # PART 3: The Dense Voting Classifier!
    # Destroy all 2D Matrix structures and Flatten into a 1D Python list.
    Flatten(),
    # Create 128 Artificial Neurons to look at the list of features!
    Dense(128, activation='relu'),
    # Randomly kill 50% of the connections natively to prevent overfitting!
    Dropout(0.5),
    # The Output! 10 Neurons mapped specifically to percentages (Softmax)
    Dense(10, activation='softmax')
])

model.summary()
# Output:
# Model: "sequential"
# ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━┓
# ┃ Layer (type)                    ┃ Output Shape           ┃       Param # ┃
# ┡━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━┩
# │ conv2d (Conv2D)                 │ (None, 30, 30, 32)     │           896 │
# ├─────────────────────────────────┼────────────────────────┼───────────────┤
# │ max_pooling2d (MaxPooling2D)    │ (None, 15, 15, 32)     │             0 │
# ├─────────────────────────────────┼────────────────────────┼───────────────┤
# │ conv2d_1 (Conv2D)               │ (None, 13, 13, 64)     │        18,496 │
# ├─────────────────────────────────┼────────────────────────┼───────────────┤
# │ max_pooling2d_1 (MaxPooling2D)  │ (None, 6, 6, 64)       │             0 │
# ├─────────────────────────────────┼────────────────────────┼───────────────┤
# │ flatten (Flatten)               │ (None, 2304)           │             0 │
# ├─────────────────────────────────┼────────────────────────┼───────────────┤
# │ dense (Dense)                   │ (None, 128)            │       295,040 │
# ├─────────────────────────────────┼────────────────────────┼───────────────┤
# │ dropout (Dropout)               │ (None, 128)            │             0 │
# ├─────────────────────────────────┼────────────────────────┼───────────────┤
# │ dense_1 (Dense)                 │ (None, 10)             │         1,290 │
# └─────────────────────────────────┴────────────────────────┴───────────────┘
#  Total params: 315,722 (1.20 MB)
#  Trainable params: 315,722 (1.20 MB)
#  Non-trainable params: 0 (0.00 B)
# 2026-03-09 23:51:33.688255: I tensorflow/core/platform/cpu_feature_guard.cc:210] This TensorFlow binary is optimized to use available CPU instructions in performance-critical operations.
# To enable the following instructions: AVX2 FMA, in other operations, rebuild TensorFlow with the appropriate compiler flags.
# /home/pollmix/Coding/AI-Engineer-Bootcamp/.venv/lib/python3.12/site-packages/keras/src/layers/convolutional/base_conv.py:113: UserWarning: Do not pass an `input_shape`/`input_dim` argument to a layer. When using Sequential models, prefer using an `Input(shape)` object as the first layer in the model instead.
#   super().__init__(activity_regularizer=activity_regularizer, **kwargs)
# WARNING: All log messages before absl::InitializeLog() is called are written to STDERR
# I0000 00:00:1773114697.173830  742492 gpu_device.cc:2020] Created device /job:localhost/replica:0/task:0/device:GPU:0 with 6082 MB memory:  -> device: 0, name: NVIDIA GeForce RTX 2080 SUPER, pci bus id: 0000:05:00.0, compute capability: 7.5
# 2026-03-09 23:51:37.569806: W external/local_xla/xla/service/gpu/llvm_gpu_backend/default/nvptx_libdevice_path.cc:41] Can't find libdevice directory ${CUDA_DIR}/nvvm/libdevice. This may result in compilation or runtime failures, if the program we try to run uses routines from libdevice.
# Searched for CUDA in the following directories:
#   ./cuda_sdk_lib
#   
# import os
# import matplotlib
# matplotlib.use('Agg')
# import matplotlib.pyplot as plt
# def _custom_show(*args, **kwargs):
#     plt.savefig('/home/pollmix/Coding/AI-Engineer-Bootcamp/docs/images/plot_b9a36286.png')
#     print('__PLOT_SAVED__')
# plt.show = _custom_show
# 
# # day4_ex.py
# from tensorflow.keras.models import Sequential
# from tensorflow.keras.layers import Conv2D, MaxPooling2D, Flatten, Dense, Dropout
# 
# # 1. Initialize the Keras Architecture
# model = Sequential([
#     # PART 1: The First Feature Extraction Block
#     # 32 Different sliding windows (kernels) processing 3-Color Channel input!
#     Conv2D(32, (3, 3), activation='relu', input_shape=(32,32,3)),
#     # Compress the (32x32) image down into a (16x16) array of brightest features!
#     MaxPooling2D((2, 2)),
#     
#     # PART 2: The Deep Feature Extraction Block
#     # Look deeper! Now use 64 different sliding windows on the compressed data!
#     Conv2D(64, (3, 3), activation='relu'),
#     # Compress the (16x16) data down into a tiny (8x8) array!
#     MaxPooling2D((2, 2)),
#     
#     # PART 3: The Dense Voting Classifier!
#     # Destroy all 2D Matrix structures and Flatten into a 1D Python list.
#     Flatten(),
#     # Create 128 Artificial Neurons to look at the list of features!
#     Dense(128, activation='relu'),
#     # Randomly kill 50% of the connections natively to prevent overfitting!
#     Dropout(0.5),
#     # The Output! 10 Neurons mapped specifically to percentages (Softmax)
#     Dense(10, activation='softmax')
# ])
# 
# model.summary().runfiles/cuda_nvcc
#   
# import os
# import matplotlib
# matplotlib.use('Agg')
# import matplotlib.pyplot as plt
# def _custom_show(*args, **kwargs):
#     plt.savefig('/home/pollmix/Coding/AI-Engineer-Bootcamp/docs/images/plot_b9a36286.png')
#     print('__PLOT_SAVED__')
# plt.show = _custom_show
# 
# # day4_ex.py
# from tensorflow.keras.models import Sequential
# from tensorflow.keras.layers import Conv2D, MaxPooling2D, Flatten, Dense, Dropout
# 
# # 1. Initialize the Keras Architecture
# model = Sequential([
#     # PART 1: The First Feature Extraction Block
#     # 32 Different sliding windows (kernels) processing 3-Color Channel input!
#     Conv2D(32, (3, 3), activation='relu', input_shape=(32,32,3)),
#     # Compress the (32x32) image down into a (16x16) array of brightest features!
#     MaxPooling2D((2, 2)),
#     
#     # PART 2: The Deep Feature Extraction Block
#     # Look deeper! Now use 64 different sliding windows on the compressed data!
#     Conv2D(64, (3, 3), activation='relu'),
#     # Compress the (16x16) data down into a tiny (8x8) array!
#     MaxPooling2D((2, 2)),
#     
#     # PART 3: The Dense Voting Classifier!
#     # Destroy all 2D Matrix structures and Flatten into a 1D Python list.
#     Flatten(),
#     # Create 128 Artificial Neurons to look at the list of features!
#     Dense(128, activation='relu'),
#     # Randomly kill 50% of the connections natively to prevent overfitting!
#     Dropout(0.5),
#     # The Output! 10 Neurons mapped specifically to percentages (Softmax)
#     Dense(10, activation='softmax')
# ])
# 
# model.summary().runfiles/cuda_nvdisasm
#   
# import os
# import matplotlib
# matplotlib.use('Agg')
# import matplotlib.pyplot as plt
# def _custom_show(*args, **kwargs):
#     plt.savefig('/home/pollmix/Coding/AI-Engineer-Bootcamp/docs/images/plot_b9a36286.png')
#     print('__PLOT_SAVED__')
# plt.show = _custom_show
# 
# # day4_ex.py
# from tensorflow.keras.models import Sequential
# from tensorflow.keras.layers import Conv2D, MaxPooling2D, Flatten, Dense, Dropout
# 
# # 1. Initialize the Keras Architecture
# model = Sequential([
#     # PART 1: The First Feature Extraction Block
#     # 32 Different sliding windows (kernels) processing 3-Color Channel input!
#     Conv2D(32, (3, 3), activation='relu', input_shape=(32,32,3)),
#     # Compress the (32x32) image down into a (16x16) array of brightest features!
#     MaxPooling2D((2, 2)),
#     
#     # PART 2: The Deep Feature Extraction Block
#     # Look deeper! Now use 64 different sliding windows on the compressed data!
#     Conv2D(64, (3, 3), activation='relu'),
#     # Compress the (16x16) data down into a tiny (8x8) array!
#     MaxPooling2D((2, 2)),
#     
#     # PART 3: The Dense Voting Classifier!
#     # Destroy all 2D Matrix structures and Flatten into a 1D Python list.
#     Flatten(),
#     # Create 128 Artificial Neurons to look at the list of features!
#     Dense(128, activation='relu'),
#     # Randomly kill 50% of the connections natively to prevent overfitting!
#     Dropout(0.5),
#     # The Output! 10 Neurons mapped specifically to percentages (Softmax)
#     Dense(10, activation='softmax')
# ])
# 
# model.summary().runfiles/nvidia_nvshmem
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
```

When you print `model.summary()`, look closely at the **Output Shape**. 
1. Our image started as $32 \times 32$.
2. The first `Conv2D` shaved off the outer edge pixels, making it $30 \times 30$.
3. The first `MaxPooling2D` cut that cleanly into $15 \times 15$.
4. The second `Conv2D` shaved more edge pixels making it $13 \times 13$.
5. The second `MaxPooling2D` cut that into a minute $6 \times 6$ image!

By the time the data reached our `Dense` layer, it was completely unrecognizable to a human being, containing only intense blocks of math highlighting distinct features of a "Frog" vs a "Dog".

## The Compiler
Because Keras is built on top of TensorFlow, compiling and training the network requires just three lines of code using the `categorical_crossentropy` Loss Function and the `Adam` mathematical optimization engine!

```python
model.compile(
    optimizer='adam',
    loss='categorical_crossentropy',
    metrics=['accuracy']
)

# Train the Convolution Scanner!
model.fit(
    X_train, y_train,
    epochs=10,
    batch_size=64,
    validation_split=0.2
)
# Output:
# Traceback (most recent call last):
#   ...
# NameError: name 'model' is not defined
```

## Wrapping Up Day 4
You have successfully built an industry-standard image classification model! 
Keras makes development blazingly fast by completely masking the underlying matrices from you. You literally construct Lego blocks, and TensorFlow seamlessly handles mathematical dimensionality behind the scenes.

But you don't control the math.

Tomorrow on **Day 5: CNNs in PyTorch**, we will rewrite this massive pipeline natively and take explicit absolute control over the data flow variables!
