FROM nvidia/cuda:12.1.1-devel-ubuntu22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV PATH=/opt/conda/bin:$PATH
ENV CONDA_PLUGINS_AUTO_ACCEPT_TOS=yes

# --- System dependencies ---
RUN apt-get update && apt-get install -y --no-install-recommends \
    wget curl git ca-certificates build-essential \
    # NVIDIA Video Codec SDK components
    libnvidia-decode-525 libnvidia-encode-525 \
    # FFmpeg and development libraries for video decoding
    ffmpeg libavcodec-dev libavformat-dev libavutil-dev libswscale-dev \
    libavfilter-dev libswresample-dev \
    # CUDA development tools for proper GPU integration
    cuda-nvcc-12-1 cuda-cudart-dev-12-1 \
    && rm -rf /var/lib/apt/lists/*
ENV LD_LIBRARY_PATH=/usr/local/cuda/lib64:${LD_LIBRARY_PATH}

# --- Verify FFmpeg GPU capabilities (non-blocking) ---
RUN ffmpeg -decoders | grep nvdec || true
RUN ffmpeg -encoders | grep nvenc || true

# --- Install Miniconda ---
RUN wget -q https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh && \
    bash miniconda.sh -b -p /opt/conda && rm miniconda.sh && \
    conda config --set always_yes yes --set changeps1 no && \
    conda update -q conda

RUN conda install -y python=3.10
#RUN conda create -n videohash python=3.10 -y
#SHELL ["conda", "run", "--no-capture-output", "-n", "videohash", "/bin/bash", "-c"]


# --- Install packages via Conda and pip ---
RUN conda install -y -c conda-forge \
    pandas \
    pyarrow \
    pillow \
    pip && \
    # Install PyTorch with CUDA 12.1 support
    pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121 && \
    # Install DALI with CUDA 12.0 support and dependencies
    pip install --extra-index-url https://pypi.nvidia.com nvidia-dali-cuda120 \
    # GPU monitoring library (required by DALI examples)
    pynvml==11.0.0 \
    # Additional packages
    imagehash opencv-contrib-python

# --- Verify DALI GPU detection (non-blocking) ---
RUN python -c "import nvidia.dali.fn as fn; import nvidia.dali as dali; print(f'DALI GPU device count: {dali.get_device_count()}')" || true



# --- Optional: working directory ---
WORKDIR /workspace
