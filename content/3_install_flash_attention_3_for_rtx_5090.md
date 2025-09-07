# Install Flash Attention 3 for RTX 5090

1. Use a cu128 PyTorch (nightly or new release variant that supports Blackwell):
```sh
# Nightly cu128 channel (Linux)
pip install --pre torch torchvision --index-url https://download.pytorch.org/whl/nightly/cu128
```

2. Build flash-attn from source with sm_120 enabled:
```sh
pip uninstall -y flash-attn flash_attn
export TORCH_CUDA_ARCH_LIST="8.0;8.6;8.9;9.0;12.0"
pip install --no-build-isolation --no-cache-dir "git+https://github.com/Dao-AILab/flash-attention.git"
```