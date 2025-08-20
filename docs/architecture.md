# fastText Architecture

This document provides a comprehensive overview of the fastText library architecture, including its core components, interfaces, and data flow.

## Overview

fastText is a library for efficient learning of word representations and sentence classification. The architecture is designed with modularity and performance in mind, featuring a high-performance C++ core with multiple language bindings and deployment targets.

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                   fastText                                       │
│                            Library Architecture                                  │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                                 INTERFACES                                      │
├─────────────────┬─────────────────┬─────────────────┬─────────────────────────┤
│   Command Line  │  Python Module  │  WebAssembly    │    C++ Library API      │
│     Interface   │    (pybind11)   │   (Emscripten)  │     (Direct Access)     │
│                 │                 │                 │                         │
│  ./fasttext     │  import         │  FastTextModule │  #include "fasttext.h"  │
│  skipgram       │  fasttext       │  (Browser/Node) │  FastText fasttext;     │
│  supervised     │  train_*()      │                 │                         │
│  predict        │  load_model()   │                 │                         │
│  quantize       │  predict()      │                 │                         │
└─────────────────┴─────────────────┴─────────────────┴─────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              CORE C++ LIBRARY                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐             │
│  │   FastText      │    │   AutoTune      │    │     Meter       │             │
│  │   (fasttext.*)  │    │   (autotune.*)  │    │   (meter.*)     │             │
│  │                 │    │                 │    │                 │             │
│  │ • Main API      │    │ • Hyperparameter│    │ • Progress      │             │
│  │ • Model loading │    │   optimization  │    │   tracking      │             │
│  │ • Training      │    │ • Grid search   │    │ • Metrics       │             │
│  │ • Prediction    │    │                 │    │                 │             │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘             │
│           │                                                                     │
│           ▼                                                                     │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐             │
│  │     Model       │    │   Dictionary    │    │      Args       │             │
│  │   (model.*)     │    │ (dictionary.*)  │    │    (args.*)     │             │
│  │                 │    │                 │    │                 │             │
│  │ • Neural network│    │ • Vocabulary    │    │ • Configuration │             │
│  │ • Forward pass  │    │ • Subword info  │    │ • Parameters    │             │
│  │ • Backward pass │    │ • Hashing       │    │ • Validation    │             │
│  │ • State mgmt    │    │ • N-grams       │    │                 │             │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘             │
│           │                       │                                             │
│           ▼                       ▼                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐             │
│  │      Loss       │    │     Matrix      │    │     Vector      │             │
│  │    (loss.*)     │    │   (matrix.*)    │    │   (vector.*)    │             │
│  │                 │    │                 │    │                 │             │
│  │ • Softmax       │    │ • DenseMatrix   │    │ • Operations    │             │
│  │ • Hierarchical  │    │ • QuantMatrix   │    │ • Utilities     │             │
│  │ • Neg. sampling │    │ • Operations    │    │                 │             │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘             │
│                                   │                                             │
│                                   ▼                                             │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐             │
│  │ ProductQuantizer│    │     Utils       │    │    Real/Types   │             │
│  │ (productquant*) │    │   (utils.*)     │    │    (real.h)     │             │
│  │                 │    │                 │    │                 │             │
│  │ • Compression   │    │ • File I/O      │    │ • Type aliases  │             │
│  │ • Quantization  │    │ • Math utils    │    │ • Constants     │             │
│  │ • Size reduction│    │ • Threading     │    │                 │             │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘             │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                           SPECIALIZED COMPONENTS                                │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐             │
│  │   Alignment     │    │  WebAssembly    │    │   Documentation │             │
│  │  (alignment/)   │    │ (webassembly/)  │    │   (website/)    │             │
│  │                 │    │                 │    │                 │             │
│  │ • Cross-lingual │    │ • Browser build │    │ • Docusaurus    │             │
│  │ • Supervised    │    │ • Emscripten    │    │ • Tutorials     │             │
│  │ • Unsupervised  │    │ • WASM bindings │    │ • API docs      │             │
│  │ • Multi-align   │    │                 │    │                 │             │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘             │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              BUILD SYSTEMS                                      │
├─────────────────────────────────────────────────────────────────────────────────┤
│                                                                                 │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐             │
│  │    Makefile     │    │   CMakeLists    │    │   Python Setup  │             │
│  │                 │    │                 │    │   (setup.py)    │             │
│  │ • Native build  │    │ • Cross-platform│    │                 │             │
│  │ • Optimized     │    │ • Library build │    │ • pip install   │             │
│  │ • Development   │    │ • Installation  │    │ • pybind11      │             │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘             │
│                                                                                 │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## Component Details

### Core C++ Components

#### FastText (fasttext.{h,cc})
- **Purpose**: Main library interface and orchestration
- **Key Responsibilities**:
  - Model training coordination
  - Loading and saving models
  - Prediction and inference
  - API surface for all operations
- **Key Methods**: `train()`, `loadModel()`, `predict()`, `getWordVector()`

#### Model (model.{h,cc})
- **Purpose**: Neural network implementation
- **Key Responsibilities**:
  - Forward and backward propagation
  - Model state management
  - Integration with loss functions
- **Architecture**: Input embeddings → Hidden layer → Output layer

#### Dictionary (dictionary.{h,cc})
- **Purpose**: Vocabulary and subword management
- **Key Responsibilities**:
  - Word-to-ID mapping
  - Subword generation (character n-grams)
  - Hash-based vocabulary storage
  - Text preprocessing

#### Matrix Operations
- **DenseMatrix**: Full precision matrices for embeddings
- **QuantMatrix**: Compressed matrices for deployment
- **ProductQuantizer**: Advanced compression algorithms

#### Loss Functions (loss.{h,cc})
- **Softmax**: Standard classification loss
- **Hierarchical Softmax**: Efficient for large vocabularies
- **Negative Sampling**: Skip-gram training optimization

### Interface Layer

#### Command Line Interface (main.cc)
- Training: `fasttext skipgram/cbow/supervised`
- Inference: `fasttext predict/predict-prob`
- Utilities: `fasttext quantize/test/dump`

#### Python Bindings (python/)
- **Implementation**: pybind11-based bindings
- **API**: Pythonic interface with `train_unsupervised()`, `train_supervised()`
- **Integration**: NumPy array support

#### WebAssembly (webassembly/)
- **Target**: Browser and Node.js environments
- **Build**: Emscripten toolchain
- **Features**: Full inference capabilities

### Specialized Components

#### Cross-lingual Alignment (alignment/)
- **Supervised**: Uses bilingual dictionaries
- **Unsupervised**: No supervision required
- **Multi-alignment**: Align multiple languages to common space

## Data Flow

### Training Pipeline
```
Input Text → Dictionary (tokenization) → Model (embeddings) → Loss (optimization) → Output Vectors
     ↓              ↓                         ↓                    ↓
  Subwords    Word/Subword IDs         Forward Pass         Gradient Updates
```

### Inference Pipeline
```
Query Text → Dictionary (lookup) → Model (forward) → Predictions/Vectors
     ↓              ↓                    ↓               ↓
   Tokens      Embedding IDs      Hidden State      Final Output
```

## Use Cases

### 1. Word Vector Learning
- **Models**: Skip-gram, CBOW
- **Features**: Subword information, character n-grams
- **Output**: Dense vector representations

### 2. Text Classification
- **Training**: Supervised learning with labeled data
- **Features**: Word n-grams, hierarchical softmax
- **Output**: Class probabilities

### 3. Language Identification
- **Models**: Pre-trained classifiers
- **Input**: Raw text
- **Output**: Language predictions

### 4. Cross-lingual Applications
- **Alignment**: Map embeddings across languages
- **Transfer**: Use models across different languages

## Build and Deployment

### Development Build
```bash
make opt          # Optimized build
make debug        # Debug build
make coverage     # Coverage analysis
```

### Production Deployment
```bash
cmake ..          # Cross-platform build
make install      # System installation
pip install .     # Python package
```

### Web Deployment
```bash
make wasm         # WebAssembly build
```

## Performance Characteristics

- **Training**: Multi-threaded, optimized linear algebra
- **Memory**: Quantization for reduced footprint
- **Speed**: Sublinear inference with hierarchical softmax
- **Scalability**: Handles large vocabularies efficiently

## Extension Points

- **Custom Loss Functions**: Implement new loss classes
- **New Algorithms**: Add to the model architecture
- **Language Support**: Extend dictionary for new languages
- **Deployment Targets**: Add new platform bindings