```
# reference: https://github.com/pytorch/ao/blob/main/torchao/_models/llama/benchmarks.sh
# Copyright (c) Meta Platforms, Inc. and affiliates.
# All rights reserved.
#
# This source code is licensed under the BSD 3-Clause license found in the
# LICENSE file in the root directory of this source tree.
export CHECKPOINT_PATH=checkpoints # path to checkpoints folder

# README BENCHMARKS
export MODEL_REPO=unsloth/Meta-Llama-3.1-8B

# baseline
python generate.py --checkpoint_path $CHECKPOINT_PATH/$MODEL_REPO/model.pth --compile --write_result benchmark_results.txt

# fp8 weights only
python generate.py --checkpoint_path $CHECKPOINT_PATH/$MODEL_REPO/model.pth --compile --quantization float8wo --write_result benchmark_results.txt

# fp8 dynamic quantization, still weights only: tensor-wise scaling
python generate.py --checkpoint_path $CHECKPOINT_PATH/$MODEL_REPO/model.pth --compile --quantization float8dq-tensor --write_result benchmark_results.txt

# fp8 dq, still wo: row-wise scaling factor
python generate.py --checkpoint_path $CHECKPOINT_PATH/$MODEL_REPO/model.pth --compile --quantization float8dq-row --write_result benchmark_results.txt
```

Use git sparse checkout for development of independent problems:
```
# Initialize sparse checkout in "cone mode" (simpler, folder-based)
git sparse-checkout init --cone

# Tell Git: "I only want to see these folders"
git sparse-checkout set torchao_float8 common_utils
```