# GPT-NeoX + ppc64le = ❤️

This README is a collection of notes I took while installing PyTorch and GPT-NeoX on ppc64le architecture (Lassen).  

Eric Hallahan sent me a really good summary of previous efforts that were made to get a stripped-down version of GPT-NeoX up and running on Summit. Most of this README has been cannibalized and grown from his work (https://github.com/EleutherAI/gpt-neox/pull/456).  

## GPT-NeoX Requirements
six  
regex  
numpy>=1.20.0  
git+git://github.com/willzywiec/DeeperSpeed.git#egg=deepspeed
transformers>=4.5.0  
tokenizers>=0.10.0  
lm_dataformat>=0.0.19  
ftfy>=6.0.0  
~~lm_eval~~  

## DeeperSpeed Requirements
mpi4py  

## Dependencies
PyTorch --> Must install from source (hard) --> **DONE**  
~~Triton --> Must install from source (easy) --> **IN PROGRESS**~~ **ABANDONED!**  
~~Eigen~~  
~~DyNet --> Must install from source (hard)~~  
~~PyArrow~~  

## Notes
* I installed PyTorch 1.8 (Python 3.8, CUDA 11.2) from Oregon State University's Open Source Lab without too many problems (https://osuosl.org/services/powerdev/opence/). I did encounter an SSL certificate error, which I was able to fix (https://stackoverflow.com/questions/31729076/conda-ssl-error).
* I unsuccessfully tried to install Triton (https://github.com/openai/triton). The output included several C++17 errors, which may have been generated by an "out-of-date" version of gcc 4.9.3 (https://stackoverflow.com/questions/60336940/g-error-unrecognized-std-c17-what-is-g-version-and-how-to-install). I installed a newer local version of gcc, but I'll have to come back to this later.
* I ignored lm_eval and its dependencies and stepped through installing everything else. The build unsuccessfully tried to install mpi4py 3.0.3, so I manually installed mpi4py 3.1.3 from conda-forge and commented the version out of the GPT-NeoX requirements.
* I received the following error when downloading the Enron email corpus: ImportError: /usr/tce/packages/gcc/gcc-4.9.3/gnu/lib64/libstdc++.so.6: version `CXXABI_1.3.11' not found. This pointed toward gcc, so I poked around a bit and found that CUDA 11.2 was missing several lib files. This led to other permission-related software installation and dependency issues, so I ended up having to uninstall everything and rebuild PyTorch 1.7.1 (Python 3.8, CUDA 10.2) from an older OSU mirror (https://ftp.osuosl.org/pub/open-ce/1.1.1/) to get things working.
* TensorboardX and Weights & Biases also weren't working, so I went through and commented those out of Megatron.

## Errors (Work In Progress)
Error when running python megatron/fused_kernels/setup.py install. The system currently uses GCC 4.9.3, so I built a local version of GCC 8.5.0.

```
                               !! WARNING !!

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
Your compiler (c++ 4.9.3) may be ABI-incompatible with PyTorch!
Please use a compiler that is ABI-compatible with GCC 5.0 and above.
See https://gcc.gnu.org/onlinedocs/libstdc++/manual/abi.html.

See https://gist.github.com/goldsborough/d466f43e8ffc948ff92de7486c5216d6
for instructions on how to install GCC 5 or higher.
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

                              !! WARNING !!
```

Error when running python deepy.py train.py -d configs small.yml local_setup.yml. TensorboardX and Weights & Biases issues...

```
File "deepy.py", line 19, in <module>
    import deepspeed
  File "/usr/workspace/zywiec1/anaconda3/envs/opence_env/lib/python3.8/site-packages/deepspeed/__init__.py", line 9, in <module>
    from .runtime.engine import DeepSpeedEngine
  File "/usr/workspace/zywiec1/anaconda3/envs/opence_env/lib/python3.8/site-packages/deepspeed/runtime/engine.py", line 17, in <module>
    from tensorboardX import SummaryWriter
  File "/usr/workspace/zywiec1/anaconda3/envs/opence_env/lib/python3.8/site-packages/tensorboardX/__init__.py", line 5, in <module>
    from .torchvis import TorchVis
  File "/usr/workspace/zywiec1/anaconda3/envs/opence_env/lib/python3.8/site-packages/tensorboardX/torchvis.py", line 11, in <module>
    from .writer import SummaryWriter
  File "/usr/workspace/zywiec1/anaconda3/envs/opence_env/lib/python3.8/site-packages/tensorboardX/writer.py", line 18, in <module>
    from .event_file_writer import EventFileWriter
  File "/usr/workspace/zywiec1/anaconda3/envs/opence_env/lib/python3.8/site-packages/tensorboardX/event_file_writer.py", line 28, in <module>
    from .proto import event_pb2
  File "/usr/workspace/zywiec1/anaconda3/envs/opence_env/lib/python3.8/site-packages/tensorboardX/proto/event_pb2.py", line 7, in <module>
    from google.protobuf import descriptor as _descriptor
  File "/usr/workspace/zywiec1/anaconda3/envs/opence_env/lib/python3.8/site-packages/google/protobuf/descriptor.py", line 47, in <module>
    from google.protobuf.pyext import _message
AttributeError: module 'google.protobuf.internal.containers' has no attribute 'MutableMapping'
```

The default version from OSU is protobuf 3.9.2, which throws a different error. The 'MutableMapping' error occurred when I upgraded to protobuf 3.19.1. I went through and commented out tensorboardX and wandb import statements in the megatron folder to fix these errors.

Error when running python deepy.py train.py -d configs small.yml local_setup.yml. Had to install best-download from conda-forge (note 'dash' and not 'underscore').

```
Traceback (most recent call last):
  File "train.py", line 20, in <module>
    from megatron.training import pretrain
  File "/usr/WS1/zywiec1/gpt-neox/megatron/training.py", line 59, in <module>
Traceback (most recent call last):
  File "train.py", line 20, in <module>
    from eval_tasks import run_eval_harness
  File "/usr/WS1/zywiec1/gpt-neox/eval_tasks/__init__.py", line 1, in <module>
    from megatron.training import pretrain
  File "/usr/WS1/zywiec1/gpt-neox/megatron/training.py", line 59, in <module>
Traceback (most recent call last):
  File "train.py", line 20, in <module>
    from .eval_adapter import EvalHarnessAdapter, run_eval_harness
  File "/usr/WS1/zywiec1/gpt-neox/eval_tasks/eval_adapter.py", line 2, in <module>
    from eval_tasks import run_eval_harness
  File "/usr/WS1/zywiec1/gpt-neox/eval_tasks/__init__.py", line 1, in <module>
    from megatron.training import pretrain
  File "/usr/WS1/zywiec1/gpt-neox/megatron/training.py", line 59, in <module>
Traceback (most recent call last):
  File "train.py", line 20, in <module>
    import best_download
ModuleNotFoundError: No module named 'best_download'
    from .eval_adapter import EvalHarnessAdapter, run_eval_harness
  File "/usr/WS1/zywiec1/gpt-neox/eval_tasks/eval_adapter.py", line 2, in <module>
    from eval_tasks import run_eval_harness
  File "/usr/WS1/zywiec1/gpt-neox/eval_tasks/__init__.py", line 1, in <module>
    from megatron.training import pretrain
  File "/usr/WS1/zywiec1/gpt-neox/megatron/training.py", line 59, in <module>
    import best_download
ModuleNotFoundError:     No module named 'best_download'from .eval_adapter import EvalHarnessAdapter, run_eval_harness

  File "/usr/WS1/zywiec1/gpt-neox/eval_tasks/eval_adapter.py", line 2, in <module>
    from eval_tasks import run_eval_harness
  File "/usr/WS1/zywiec1/gpt-neox/eval_tasks/__init__.py", line 1, in <module>
    import best_download
ModuleNotFoundError: No module named 'best_download'
    from .eval_adapter import EvalHarnessAdapter, run_eval_harness
  File "/usr/WS1/zywiec1/gpt-neox/eval_tasks/eval_adapter.py", line 2, in <module>
    import best_download
ModuleNotFoundError: No module named 'best_download'
```

## Code Snippets
Initial torch test shown below on one debug node

```
Python 3.8.12 (default, Oct 12 2021, 13:02:29)
[GCC 7.3.0] :: Anaconda, Inc. on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import torch
>>> torch.cuda.is_available()
True
>>> torch.cuda.current_device()
0
>>> torch.cuda.device(0)
<torch.cuda.device object at 0x2000004de370>
>>> torch.cuda.device_count()
4
>>> torch.cuda.get_device_name(0)
'Tesla V100-SXM2-16GB'
>>>
```

Output from conda list

```
# Name                    Version                   Build  Channel
_libgcc_mutex             0.1                        main
_pytorch_select           2.0                  cuda10.2_1    https://ftp.osuosl.org/pub/open-ce/1.1.1
absl-py                   0.10.0                   py38_0
aiohttp                   3.8.1            py38h140841e_0
aiosignal                 1.2.0              pyhd3eb1b0_0
async-timeout             4.0.1              pyhd3eb1b0_0
attrs                     21.2.0             pyhd3eb1b0_0
av                        8.0.2            py38h049efb5_1    https://ftp.osuosl.org/pub/open-ce/1.1.1
blas                      1.0                    openblas
blinker                   1.4              py38h6ffa863_0
blosc                     1.21.0               h5f94dde_0
brotli                    1.0.9                he6710b0_2
brotlipy                  0.7.0           py38h140841e_1003
brunsli                   0.1                  h29c3540_0
bzip2                     1.0.8                h7b6447c_0
c-ares                    1.17.1               h140841e_0
ca-certificates           2021.10.26           h6ffa863_2
cachetools                4.2.2              pyhd3eb1b0_0
certifi                   2021.10.8        py38hf8b3453_1    conda-forge
cffi                      1.14.6           py38hf9d8e4b_0
cfitsio                   3.470                hf0d0db6_6
chardet                   3.0.4           py38h6ffa863_1003
charls                    2.2.0                h29c3540_0
charset-normalizer        2.0.4              pyhd3eb1b0_0
click                     7.0                        py_0
cloudpickle               2.0.0              pyhd3eb1b0_0
configparser              5.2.0                    pypi_0    pypi
cryptography              36.0.0           py38h179485c_0
cudatoolkit               10.2.89              hfd86e86_1
cudnn                     7.6.5_10.2           h9286eec_2    https://ftp.osuosl.org/pub/open-ce/1.1.1
cycler                    0.11.0             pyhd3eb1b0_0
cytoolz                   0.11.0           py38h7b6447c_0
dask-core                 2021.10.0          pyhd3eb1b0_0
decorator                 5.1.0              pyhd3eb1b0_0
deepspeed                 0.3.15+b250d97           pypi_0    pypi
docker-pycreds            0.4.0                    pypi_0    pypi
einops                    0.3.2                    pypi_0    pypi
ffmpeg                    4.2                  h1a5d6f3_0
filelock                  3.4.0                    pypi_0    pypi
fonttools                 4.25.0             pyhd3eb1b0_0
freetype                  2.11.0               h9215f1b_0
frozenlist                1.2.0            py38h140841e_0
fsspec                    2021.10.1          pyhd3eb1b0_0
ftfy                      6.0.3                    pypi_0    pypi
future                    0.18.2                   py38_1
giflib                    5.2.1                h7b6447c_0
gitdb                     4.0.9                    pypi_0    pypi
gitpython                 3.1.24                   pypi_0    pypi
gmp                       6.2.1                h29c3540_0
google-auth               1.23.0             pyhd3eb1b0_0
google-auth-oauthlib      0.4.4              pyhd3eb1b0_0
grpcio                    1.31.0           py38hf8bcb03_0
huggingface-hub           0.2.1                    pypi_0    pypi
idna                      2.8                      py38_0
imagecodecs               2021.8.26        py38h74076e2_0
imageio                   2.9.0              pyhd3eb1b0_0
joblib                    1.1.0                    pypi_0    pypi
jpeg                      9d                   h140841e_0
jsonlines                 3.0.0                    pypi_0    pypi
jxrlib                    1.1                  h7b6447c_2
kiwisolver                1.3.1            py38h29c3540_0
krb5                      1.19.2               h6205695_0
lame                      3.100                h7b6447c_0
lcms2                     2.12                 h2045e0b_0
ld_impl_linux-ppc64le     2.33.1               h0f24833_7
lerc                      3.0                  h29c3540_0
leveldb                   1.20                 hf484d3e_1
libaec                    1.0.4                he6710b0_1
libcurl                   7.80.0               ha47bf17_0
libdeflate                1.8                  h140841e_5
libedit                   3.1.20210910         h140841e_0
libev                     4.33                 h140841e_1
libffi                    3.3                  he6710b0_2
libgcc-ng                 8.2.0                h822a55f_1
libgfortran-ng            7.3.0                h822a55f_1
libnghttp2                1.46.0               hedb86c2_0
libopenblas               0.3.13               h989ec91_0
libopus                   1.3.1                h7b6447c_0
libpng                    1.6.37               hbc83047_0
libprotobuf               3.9.2                h847787d_2    https://ftp.osuosl.org/pub/open-ce/1.1.1
libssh2                   1.9.0                h1ba5d50_1
libstdcxx-ng              8.2.0                h822a55f_1
libtiff                   4.2.0                h781710b_0
libvpx                    1.7.0                hf484d3e_0
libwebp                   1.2.0                he32dc1f_0
libwebp-base              1.2.0                h140841e_0
libzopfli                 1.0.3                he6710b0_0
llvmlite                  0.31.0           py38hd408876_0
lm-dataformat             0.0.20                   pypi_0    pypi
lmdb                      0.9.29               h29c3540_0
locket                    0.2.1            py38h6ffa863_1
lz4-c                     1.9.3                h29c3540_1
markdown                  3.1.1                    py38_0
matplotlib-base           3.5.0            py38h724cb3c_0
mpi                       1.0                       mpich
mpi4py                    3.0.3            py38h028fd6f_0
mpich                     3.3.2                hc856adb_0
multidict                 5.1.0            py38h140841e_2
munkres                   1.1.4                      py_0
nccl                      2.7.8                cuda10.2_3    https://ftp.osuosl.org/pub/open-ce/1.1.1
ncurses                   6.3                  h140841e_2
networkx                  2.3                        py_0
ninja                     1.10.2.3                 pypi_0    pypi
numactl                   2.0.12               h459fe5f_2    https://ftp.osuosl.org/pub/open-ce/1.1.1
numba                     0.47.0           py38h962f231_0
numpy                     1.19.2           py38h6163131_0
numpy-base                1.19.2           py38h75fe3a5_0
oauthlib                  3.1.0                      py_0
olefile                   0.46               pyhd3eb1b0_0
onnx                      1.6.0                    py38_2    https://ftp.osuosl.org/pub/open-ce/1.1.1
openjpeg                  2.4.0                hfe35807_0
openssl                   1.1.1l               h140841e_0
packaging                 21.3               pyhd3eb1b0_0
partd                     1.2.0              pyhd3eb1b0_0
pathtools                 0.1.2                    pypi_0    pypi
pillow                    7.1.2            py38haac5956_0
pip                       21.3.1                   pypi_0    pypi
promise                   2.3                      pypi_0    pypi
protobuf                  3.19.1                   pypi_0    pypi
psutil                    5.8.0                    pypi_0    pypi
pyasn1                    0.4.8              pyhd3eb1b0_0
pyasn1-modules            0.2.8                      py_0
pybind11                  2.8.1                    pypi_0    pypi
pycparser                 2.21               pyhd3eb1b0_0
pyjwt                     1.7.1                    py38_0
pyopenssl                 21.0.0             pyhd3eb1b0_1
pyparsing                 3.0.6                    pypi_0    pypi
pysocks                   1.7.1            py38h6ffa863_0
python                    3.8.12               h836d2c2_0
python-dateutil           2.8.2              pyhd3eb1b0_0
python-lmdb               0.98             py38he6710b0_0
python_abi                3.8                      2_cp38    conda-forge
pytorch                   1.7.1                hca541ab_1    https://ftp.osuosl.org/pub/open-ce/1.1.1
pytorch-base              1.7.1           cuda10.2_py38_8    https://ftp.osuosl.org/pub/open-ce/1.1.1
pywavelets                1.1.1            py38h7b6447c_2
pyyaml                    5.4.1            py38h140841e_1
readline                  8.1                  h140841e_0
regex                     2021.11.10               pypi_0    pypi
requests                  2.22.0                   py38_1
requests-oauthlib         1.3.0                      py_0
rsa                       4.7.2              pyhd3eb1b0_1
rust                      1.54.0               h6ffa863_0
sacremoses                0.0.46                   pypi_0    pypi
scikit-image              0.17.2           py38hdf5156a_0
scipy                     1.4.1            py38habc2bb6_0
sentencepiece             0.1.91                   py38_2    https://ftp.osuosl.org/pub/open-ce/1.1.1
sentry-sdk                1.5.1                    pypi_0    pypi
setuptools                58.0.4           py38h6ffa863_0
shortuuid                 1.0.8                    pypi_0    pypi
six                       1.15.0           py38h6ffa863_0
smmap                     5.0.0                    pypi_0    pypi
snappy                    1.1.8                he6710b0_0
sqlite                    3.36.0               hd7247d8_0
subprocess32              3.5.4                    pypi_0    pypi
tabulate                  0.8.9            py38h6ffa863_0
tbb                       2021.4.0             h66086b3_0
tensorboard               2.4.0              pyhc2a3f3e_1    https://ftp.osuosl.org/pub/open-ce/1.1.1
tensorboard-plugin-wit    1.6.0              pyhc0078e9_1    https://ftp.osuosl.org/pub/open-ce/1.1.1
tensorboardx              2.2                      pypi_0    pypi
termcolor                 1.1.0                    pypi_0    pypi
tifffile                  2021.7.2           pyhd3eb1b0_2
tk                        8.6.11               h7e00dab_0
tokenizers                0.10.3                   pypi_0    pypi
toolz                     0.11.2             pyhd3eb1b0_0
torchtext                 0.8.1                    py38_4    https://ftp.osuosl.org/pub/open-ce/1.1.1
torchvision-base          0.8.2           cuda10.2_py38_5    https://ftp.osuosl.org/pub/open-ce/1.1.1
tqdm                      4.41.1                     py_0
transformers              4.15.0                   pypi_0    pypi
typing-extensions         4.0.1                    pypi_0    pypi
typing_extensions         3.10.0.2           pyh06a4308_0
ujson                     5.1.0                    pypi_0    pypi
urllib3                   1.25.11                    py_0
wandb                     0.12.9                   pypi_0    pypi
wcwidth                   0.2.5                    pypi_0    pypi
werkzeug                  0.16.0                     py_0
wheel                     0.37.0             pyhd3eb1b0_1
xz                        5.2.5                h7b6447c_0
yaml                      0.2.5                h7b6447c_0
yarl                      1.6.3            py38h140841e_0
yaspin                    2.1.0                    pypi_0    pypi
zfp                       0.5.5                h29c3540_6
zlib                      1.2.11               h140841e_4
zstandard                 0.16.0                   pypi_0    pypi
zstd                      1.4.9                hc52992f_0
```
