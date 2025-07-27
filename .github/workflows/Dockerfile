FROM nvidia/cuda:12.1.1-devel-ubuntu22.04

ENV DEBIAN_FRONTEND=noninteractive
ENV PATH=/opt/conda/bin:$PATH
ENV CONDA_PLUGINS_AUTO_ACCEPT_TOS=yes

# --- System dependencies ---
RUN apt-get update && apt-get install -y --no-install-recommends \
    wget curl git ca-certificates build-essential \
    # NVIDIA Video Codec SDK components
    libnvidia-decode-525 libnvidia-encode-525 \
    # CUDA development tools for proper GPU integration
    cuda-nvcc-12-1 cuda-cudart-dev-12-1     && rm -rf /var/lib/apt/lists/*     && apt-get clean

ENV LD_LIBRARY_PATH=/usr/local/cuda/lib64:${LD_LIBRARY_PATH}

# --- Install nv-codec-headers ---
RUN apt-get update && apt-get install -y --no-install-recommends git && \
    git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git /usr/local/src/nv-codec-headers && \
    cd /usr/local/src/nv-codec-headers && \
    make && make install &&     rm -rf /usr/local/src/nv-codec-headers

# --- Install FFmpeg build dependencies ---
RUN apt-get update && apt-get install -y --no-install-recommends \
    pkg-config yasm nasm libx264-dev libx265-dev libnuma-dev libvpx-dev \
    libfdk-aac-dev libmp3lame-dev libopus-dev libdav1d-dev libvmaf-dev \
    libass-dev libfreetype6-dev libfontconfig1-dev libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# --- Compile FFmpeg with NVDEC support (version 7) ---
RUN mkdir -p /usr/local/src/ffmpeg && \
    cd /usr/local/src/ffmpeg && \
    wget -q https://ffmpeg.org/releases/ffmpeg-7.0.1.tar.xz -O ffmpeg.tar.xz && \
    tar -xf ffmpeg.tar.xz --strip-components=1 && \
    ./configure \
        --prefix=/usr/local \
        --enable-cuda-nvcc \
        --enable-cuvid \
        --enable-nvdec \
        --enable-libnpp \
        --extra-cflags="-I/usr/local/cuda/include/ -I/usr/local/include/ffnvcodec" \
        --extra-ldflags="-L/usr/local/cuda/lib64/" \
        --disable-static \
        --enable-shared \
        --enable-gpl \
        --enable-libx264 \
        --enable-libx265 \
        --enable-libvpx \
        --enable-libfdk-aac \
        --enable-libmp3lame \
        --enable-libopus \
        --enable-libdav1d \
        --enable-libass \
        --enable-libfreetype \
        --enable-libfontconfig \
        --enable-nonfree && \
    make -j$(nproc) && \
    make install && \
    ldconfig && \
    rm -rf /usr/local/src/ffmpeg

# --- Verify FFmpeg GPU capabilities (non-blocking) ---
RUN ffmpeg -decoders | grep nvdec || true
RUN ffmpeg -encoders | grep nvenc || true

# --- Install Miniconda ---
RUN wget -q https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh && \
    bash miniconda.sh -b -p /opt/conda && rm miniconda.sh && \
    conda config --set always_yes yes --set changeps1 no && \
    conda update -q conda && \
    conda clean --all

RUN conda install -y python=3.10

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

# --- Cleanup Conda ---
RUN conda clean --all

# --- Verify DALI GPU detection (non-blocking) ---
RUN python -c "import nvidia.dali.fn as fn; import nvidia.dali as dali; print(f'DALI GPU device count: {dali.get_device_count()}')" || true

# --- Optional: working directory ---
WORKDIR /workspace
