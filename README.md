# Labrakates

Labrakates is a project to run [InstructLab](https://instructlab.ai/)
in [Kubernetes](https://kubernetes.io/) using [Tekton
Pipelines](https://tekton.dev/), enabling users to fine-tune large
language models with their own repository of knowledge or skills
within their GPU-enabled Kubernetes clusters.

The name is roughly inspired by the InstructLab labrador combined with
the way some people pronounce the Kubernetes abbreviation "k8s" as
"kates" / "keights".

# Overview

InstructLab has two distinct steps that are required for fine-tuning a
large language model - synthetic data generation and training.

## Synthetic Data Generation Pipeline

This generate pipeline generates synthetic question and answer pairs,
which will potentially be used later for training a student model,
from an [InstructLab
taxonomy](https://github.com/instructlab/taxonomy) repository. The
expected use is that you have your own taxonomy repository containing
the knowledge or skills you want to teach your student model. An
example taxonomy repositoy is maintained at
[konflux-taxonomy](https://github.com/bbrowning/konflux-taxonomy).

The output of the generate pipeline is a set of json training data
files, stored in S3-compatible object storage of your choice.

## Training Pipeline

The train pipeline takes an input set of generated data and fine-tunes
the student model of your choice, using either a full fine-tuning or
LoRA methods (to save on hardware resources at the expense of some
accuracy).

The output of the train pipeline is a set of model files (or LoRA
adapter weights) in HuggingFace repository format, stored in
S3-compatible object storage of your choice.
