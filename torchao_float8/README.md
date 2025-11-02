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
python generate.py --checkpoint_path $CHECKPOINT_PATH/$MODEL_REPO/model.pth --compile --write_result benchmark_results.txt
python generate.py --checkpoint_path $CHECKPOINT_PATH/$MODEL_REPO/model.pth --compile --quantization float8wo --write_result benchmark_results.txt
python generate.py --checkpoint_path $CHECKPOINT_PATH/$MODEL_REPO/model.pth --compile --quantization float8dq-tensor --write_result benchmark_results.txt
python generate.py --checkpoint_path $CHECKPOINT_PATH/$MODEL_REPO/model.pth --compile --quantization float8dq-wo --write_result benchmark_results.txt
```