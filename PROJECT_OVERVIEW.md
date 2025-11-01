# FringeSense: Real-Time Multi-Task Tactile Sensing System - Comprehensive Project Overview

## Executive Summary

**FringeSense** is a novel multimodal tactile sensing platform developed as a Master's Thesis project at TU Delft. The system combines high-speed vision and force sensing to deliver real-time force, position, and shape estimation, bridging advanced robotics, machine learning, and healthcare technologies. The project was created as part of a collaborative effort across three departments and resulted in a complete end-to-end pipeline from hardware data acquisition to real-time deep learning inference.

**Key Achievement:** Developed a complete tactile sensing system achieving 0.1459 N RMSE in force vector regression, sub-millimeter position acquisition accuracy, and 96.2% shape classification accuracy through real-time analysis of photoelastic fringe patterns.

**Published Thesis:** Available at [TU Delft Repository](https://resolver.tudelft.nl/uuid:012aeaa6-55b9-497d-8dd5-de783d0a5c98)

---

## Project Scope and Objectives

### Primary Goal
Design and implement a real-time tactile sensing system capable of simultaneously predicting:
1. **Force vectors** (3-axis forces + 3-axis moments: Fx, Fy, Fz, Mx, My, Mz, Ft)
2. **Contact point positions** (X, Y coordinates with sub-millimeter accuracy)
3. **Shape classification** (identifying objects through tactile feedback)

### Key Requirements
- Real-time inference at 10 Hz for live demonstrations
- High precision force sensing (<0.2 N RMSE)
- Sub-millimeter position accuracy
- Robust shape classification (>95% accuracy)
- Complete pipeline from hardware acquisition to ML inference

---

## Technical Architecture

### 1. Hardware System

#### Force Sensing
- **Device:** NI-DAQ with 6-axis force/torque sensor
- **Sampling Rate:** 5000 Hz for high-precision force measurements
- **Calibration:** Custom calibration matrix from sensor documentation
- **Channels:** 6-axis measurements (Fx, Fy, Fz, Mx, My, Mz)

#### Vision System
- **Camera:** Basler GigE/USB3 camera for tactile imaging
- **Frame Rate:** 50-500 Hz (typically 50 Hz for stable acquisition)
- **Image Type:** Photoelastic fringe patterns captured in grayscale/RGB
- **Resolution:** High-resolution TIFF format for ML training

#### Synchronization
- Precise timestamp synchronization between force sensor and camera
- Sub-millisecond timing accuracy for ground-truth labeling
- Robust handling of frame loss and timing diagnostics

### 2. Software Stack

#### Programming Languages & Core Libraries
- **Python 3.x** - Primary development language
- **PyTorch** - Deep learning framework for model training and inference
- **NumPy** - Numerical computations and array operations
- **Pandas** - Data manipulation and CSV handling
- **Matplotlib & Seaborn** - Data visualization and plotting

#### Computer Vision & Image Processing
- **OpenCV (cv2)** - Image processing and manipulation
- **PIL/Pillow** - Image loading and preprocessing
- **torchvision** - PyTorch computer vision utilities and transforms

#### Hardware Interface
- **nidaqmx** - National Instruments DAQ interface for force sensor
- **pypylon** - Basler camera Python API
- **pynput** - Mouse/keyboard input for experiment triggering

#### Scientific Computing & ML
- **scikit-learn** - ML utilities, metrics, preprocessing
- **scipy** - Signal processing (filtering, Butterworth filters)
- **tqdm** - Progress bars for long-running operations

#### Data Management & Utilities
- **csv, json** - Data serialization formats
- **threading, concurrent.futures** - Multi-threaded data acquisition
- **pathlib, shutil** - File system operations
- **logging** - Diagnostic logging and debugging

---

## Complete Pipeline Architecture

### Phase 1: Data Acquisition (`acquisition/`)

**Purpose:** Synchronized collection of force sensor and camera data from hardware

**Key Components:**
- `_00_measurement_setup.py` - Interactive experimental parameter configuration
- `_01_acquisition.py` - Hardware control classes (ForceAcquisition, CameraAcquisition)
- `_02_saving_data.py` - Structured file organization with timestamps
- `_03_analysis.py` - Quality diagnostics and frame loss analysis
- `_04_force_to_image_mapping.py` - Creates ML-ready force-to-image mappings
- `_05_low_pass_filter.py` - Signal processing and noise reduction
- `_06_plotting.py` - Data verification and visualization plots
- `_07_video_creation.py` - Presentation video generation
- `main_acquisition_pipeline.py` - Orchestrates the complete acquisition workflow

**Workflow:**
1. Interactive setup of positions, indenters, and measurement sequences
2. Synchronized data collection at 5000 Hz (force) and 50 Hz (camera)
3. Automatic file organization with timestamps
4. Frame loss diagnostics and quality control
5. Force-to-image temporal alignment for ML training
6. Optional low-pass filtering for noise reduction
7. Spatial labeling with position coordinates
8. Visualization plots for data verification

**Output:**
- `Trial_XXX/` folders containing:
  - `Images/` - Tactile fringe pattern images (TIFF)
  - `force.csv` - 6-axis force/torque measurements
  - `*_timestamps.csv` - Synchronization data
  - `Results/` - ML-ready datasets with force-image mappings

### Phase 2: Data Preprocessing (`preprocessing/`)

**Purpose:** Combine and clean raw datasets from multiple trials

**Key Components:**
- `combine_full_dataset.py` - Merges multiple trials into unified dataset

**Workflow:**
1. Load raw acquisition results from multiple trials
2. Clean data (remove invalid/duplicate entries)
3. Standardize column names and data types
4. Handle missing data appropriately
5. Merge into single ML-ready CSV

**Output:**
- `full_dataset.csv` - Unified, cleaned dataset for augmentation

### Phase 3: Data Augmentation (`augmentation/`)

**Purpose:** Expand dataset diversity for robust ML training

**Key Components:**
- `augment_images_desktop_and_super_computer.py` - Image and label augmentation

**Workflow:**
1. Load cleaned dataset
2. Apply diverse transformations:
   - Geometric transformations (rotation, scaling, translation)
   - Color/intensity augmentations
   - Noise injection
   - Masking techniques
3. Track contact points through transformations
4. Generate augmented labels with proper synchronization
5. Support both desktop and supercomputer batch processing

**Output:**
- `augmented_images/` - Augmented tactile images
- `augmented_labels.csv` - Corresponding labels with metadata

### Phase 4: Postprocessing (`postprocessing/`)

**Purpose:** Clean and standardize augmented labels for ML

**Key Components:**
- `postprocess_labels.py` - Label refinement and standardization

**Workflow:**
1. Load augmented labels
2. Convert forces to absolute values
3. Handle no-contact frames (set shape/position to None/NaN)
4. Rename columns to ML-compatible format
5. Sort by image and augmentation index
6. Remap shape labels to integer classes
7. Export final ML-ready CSV

**Output:**
- `processed_labels.csv` - Final, clean labels for training

### Phase 5: Model Training (`training/`)

**Purpose:** Train multi-task deep learning models for force, class, and position prediction

**Key Components:**
- `train_multitask_model.py` - Main training script
- `data_preparation.py` - Dataset loading and formatting
- `ResNet18_multitask.py` - ResNet18 architecture with multi-task heads
- `SimpleCNN_multitask.py` - Custom CNN architecture

**Model Architecture: IsoNet**
- **Backbone:** ResNet18 (pretrained or from scratch)
- **Three Output Heads:**
  1. **Force Head:** 7-dimensional regression (Fx, Fy, Fz, Mx, My, Mz, Ft)
  2. **Classification Head:** Shape classification (number of classes varies)
  3. **Position Head:** 2-dimensional regression (X, Y contact coordinates)

**Training Configuration:**
- Batch size: 32
- Learning rate: 1e-3
- Optimizer: Adam
- Loss functions: MSE (regression), CrossEntropy (classification)
- Data splits: 80% train, 10% validation, 10% test
- Checkpointing: Save best model based on validation loss
- Metrics tracking: RMSE, accuracy, precision, recall, F1-score

**Workflow:**
1. Load augmented images and postprocessed labels
2. Split into train/validation/test sets
3. Initialize model architecture (ResNet18 or custom CNN)
4. Train with validation monitoring
5. Save checkpoints for best model
6. Evaluate on test set
7. Generate performance plots and metrics

**Output:**
- `model_weights.pth` - Trained model checkpoint
- `training_loss_plot.png` - Training/validation curves
- `metrics_summary.json` - Complete evaluation metrics
- Confusion matrices and classification reports

### Phase 6: Real-Time Inference (`inference/`)

**Purpose:** Deploy trained models for real-time or batch prediction

**Key Components:**
- `main.py` - Unified inference script
- `inference_dataset.py` - Dataset loader for new data
- `model_ResNet18.py` - ResNet18 inference model
- `model_own_CNN.py` - Custom CNN inference model

**Inference Modes:**
- **Batch Mode:** Process folder of images
- **Live Mode:** Real-time camera feed processing at 10 Hz

**Workflow:**
1. Load trained model weights
2. Initialize camera or load batch images
3. Preprocess images (crop, mask, normalize)
4. Run inference through model
5. Post-process predictions
6. Output results (print, save CSV, visualize)

**Real-Time Performance:**
- **Inference Rate:** 10 Hz for live demonstrations
- **Latency:** <100ms per frame
- **Device Support:** CPU and CUDA GPU

**Output:**
- `predictions.csv` - Batch prediction results
- `live_results/` - Real-time inference outputs
- Visualization overlays on images

---

## Performance Metrics

### Force Vector Prediction
- **RMSE:** 0.1459 N across all 7 force/torque components
- **R² Score:** >0.95 for force regression
- **Real-time capability:** 10 Hz inference rate

### Position Acquisition
- **Accuracy:** Sub-millimeter precision (<1mm error)
- **X-Y Coordinate Prediction:** Continuous 2D output
- **Spatial Resolution:** High precision contact localization

### Shape Classification
- **Accuracy:** 96.2% on test set
- **Precision (macro):** >0.95
- **Recall (macro):** >0.95
- **F1-Score (macro):** >0.95

### System Performance
- **Data Acquisition:** 5000 Hz force sampling, 50 Hz camera
- **Dataset Size:** Large-scale multimodal dataset
- **Training Time:** Optimized for desktop and supercomputer
- **Inference Speed:** Real-time at 10 Hz on standard hardware

---

## Key Technical Innovations

1. **Multimodal Synchronization:** Precise timestamp alignment between high-speed force data (5000 Hz) and camera frames (50 Hz) for accurate ground-truth labeling

2. **Photoelastic Fringe Analysis:** Computer vision processing of photoelastic patterns to extract tactile information without physical contact with the sensor surface

3. **Multi-Task Learning:** Single neural network simultaneously predicts force vectors, contact positions, and object shapes, sharing feature representations for efficiency

4. **Real-Time Optimization:** Model architecture and preprocessing pipeline optimized for 10 Hz inference on standard hardware

5. **Robust Data Pipeline:** Complete end-to-end workflow from hardware to inference with quality control at each stage

6. **Cross-Platform Support:** Augmentation and training scripts compatible with desktop and supercomputer environments

---

## Dataset Characteristics

### Raw Data Collection
- Multiple experimental trials with systematic parameter variation
- Diverse indenters and contact scenarios
- Comprehensive coverage of force ranges and positions

### Data Quality Assurance
- Frame loss monitoring and diagnostics
- Timestamp synchronization validation
- Consistent force-image temporal alignment
- Filtering options for noise reduction

### Augmentation Strategy
- Geometric transformations preserving physical constraints
- Color and intensity variations
- Noise injection for robustness
- Contact point tracking through transformations

### Final Dataset
- Large-scale multimodal tactile sensing dataset
- Balanced representation across classes
- Train/validation/test splits with stratification
- ML-ready CSV format with comprehensive metadata

---

## Software Engineering Practices

### Code Organization
- Modular architecture with clear separation of concerns
- Consistent naming conventions and directory structure
- README files in each module for documentation
- Inline code documentation and docstrings

### Configuration Management
- User-configurable parameters in script headers
- Separate configs for desktop and supercomputer
- Path management with pathlib
- Environment-specific settings

### Error Handling
- Robust frame loss detection and recovery
- Hardware connection validation
- Data validation at each pipeline stage
- Comprehensive logging and diagnostics

### Performance Optimization
- Multi-threaded data acquisition
- Batch processing for augmentation
- GPU acceleration for training/inference
- Efficient data loading with PyTorch DataLoader

---

## Research Context

### Interdisciplinary Collaboration
- **Robotics Department:** Hardware integration and control systems
- **Machine Learning Group:** Deep learning model development
- **Healthcare Technology:** Application to medical robotics

### Applications
- Robotic surgery with tactile feedback
- Prosthetics with touch sensation
- Industrial automation with force control
- Human-robot interaction

### Future Extensions
- Multi-sensor fusion approaches
- Transfer learning to different tactile sensors
- Integration with robotic manipulation systems
- Extension to dynamic contact scenarios

---

## Technical Requirements Summary

### Development Environment
- Python 3.x
- CUDA-capable GPU (optional, for acceleration)
- Windows/Linux operating system

### Hardware Requirements (Acquisition)
- NI-DAQ device with 6-axis force sensor
- Basler GigE/USB3 camera
- High-performance workstation for data collection

### Hardware Requirements (Training/Inference)
- Modern multi-core CPU
- 16+ GB RAM recommended
- NVIDIA GPU with CUDA support (optional)
- SSD for fast data loading

### Software Dependencies
```
Core ML: torch, torchvision, scikit-learn
Data Processing: pandas, numpy, scipy
Computer Vision: opencv-python, PIL/Pillow
Hardware Interface: nidaqmx, pypylon
Visualization: matplotlib, seaborn
Utilities: tqdm, pynput, pathlib
```

---

## Project Deliverables

1. **Complete Data Acquisition System:** Hardware interface and synchronized collection pipeline
2. **ML-Ready Dataset:** Large-scale multimodal tactile sensing dataset
3. **Trained Models:** ResNet18 and custom CNN architectures with checkpoints
4. **Real-Time Inference System:** Live demonstration capability at 10 Hz
5. **Comprehensive Documentation:** READMEs, code comments, and inline documentation
6. **Master's Thesis:** Published at TU Delft repository
7. **Complete Codebase:** Modular, well-organized Python implementation

---

## Conclusion

FringeSense represents a complete tactile sensing solution, from hardware data acquisition through real-time ML inference. The project successfully demonstrates the integration of advanced computer vision, deep learning, and precision instrumentation to achieve high-performance tactile sensing suitable for robotics and healthcare applications. The modular architecture and comprehensive documentation make the system accessible for future research and development.

---

**Project Repository:** YannickSerrien/FringeSense-Master_Thesis  
**Thesis Link:** https://resolver.tudelft.nl/uuid:012aeaa6-55b9-497d-8dd5-de783d0a5c98  
**Institution:** TU Delft  
**Year:** 2025
