# GPT-NeoX + ppc64le = ❤️

This README is a collection of notes I took while installing PyTorch and GPT-NeoX on ppc64le architecture (Lassen).  

Eric Hallahan sent me a really good summary of previous efforts that were made to get a stripped-down version of GPT-NeoX up and running on ppc64le. Most of this README is cannibalized from his work (https://github.com/EleutherAI/gpt-neox/pull/456).  

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
* I unsuccessfully tried to install Triton (https://github.com/openai/triton). The output included several C++17 errors, which may have been generated by an "out-of-date" version of gcc 4.9.3 (https://stackoverflow.com/questions/60336940/g-error-unrecognized-std-c17-what-is-g-version-and-how-to-install). I installed a newer local version of gcc, but sparse attention isn't really necessary, so I abandoned this effort. I'll maybe come back to this later.
* I ignored lm_eval and its dependencies and stepped through installing everything else. The build unsuccessfully tried to install mpi4py 3.0.3, so I manually installed mpi4py 3.1.3 from conda-forge and commented the version out of the GPT-NeoX requirements.
* I received the following error when downloading the Enron email corpus: ImportError: /usr/tce/packages/gcc/gcc-4.9.3/gnu/lib64/libstdc++.so.6: version `CXXABI_1.3.11' not found. This pointed toward gcc, so I poked around a bit and found that CUDA 11.2 was missing several files. This led to other permission-related software installation and dependency issues, so I ended up having to uninstall and rebuild PyTorch 1.7.1 (Python 3.8, CUDA 10.2) from an older mirror (https://ftp.osuosl.org/pub/open-ce/1.1.1/) to get things working.
* TensorboardX appears to not be working...

## Code Snippets
Initial torch test shown below with one queued debug node.

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
