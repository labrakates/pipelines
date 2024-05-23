# Labrakates

Labrakates is a project to run [InstructLab](https://instructlab.ai/)
on top of [Kubernetes](https://kubernetes.io/) using [Tekton
Pipelines](https://tekton.dev/).

The name is roughly inspired by the InstructLab labrador combined with
the way some people pronounce the Kubernetes abbreviation "k8s" as
"kates" / "keights".

# Deployment

## Login to OpenShift w/ oc CLI

`oc login ...`

## HuggingFace Token Secret

Popular student or teacher models require `$HF_TOKEN` to be set to
download from HuggingFace Hub. Examples of models that require this
are `mistralai/Mixtral-8x7B-Instruct-v0.1`. This step may be skipped
if you are only using models that do not require a token, and you
comment out the huggingface-token workspaces from the PipelineRun
examples.

```
oc create secret generic hf-token --from-literal=hf_token=<your huggingface token>
```

## Deploying the Tekton tasks and pipelines

### Lab Tasks
`oc apply -f tasks/`

### Lab Pipelines
`oc apply -f pipelines/`

### Persistent Volume Claims (PVC)

HuggingFace cache and an ObjectBucketClaim for S3 data

`oc apply -f pvc/`

## Running the pipelines

### Start synthetic data generation

`oc create -f generate.yaml`

### Start training

After generation finishes copy the `generate-results-bucket-dir`
pipeline result value and paste it into `train.yaml` under a param of
the same name.

Then,

`oc create -f train.yaml`


## Test the newly trained model

**TODO** Fix this to download the trained model from S3 during deployment...


After a successful pipeline run, we can test the new model.

First, deploy a vLLM inference container and service:

`oc apply -f deployment.yaml`

Once that deployed pod comes up, port-forward it to your local
machine:

`oc port-forward deployment/fine-tuned-merlinite-7b 8000`

Finally, interact with your newly trained model using `ilab`. From a
new terminal:

`ilab chat -m "instructlab-taxonomy/training_results/final"`


If all went well, this will connect to your deployed vLLM model and
you can interact with it as if you trained the model locally.


# Building and pushing new container images

Instructions to rebuild the container images used in your pipeline
tasks are below. Be warned that these are large images - and can take
a long time to push if you don't have a fast upload speed.

## CLI Container

### Building the container
`podman build -f containers/ilab/Containerfile -t ilab-cli`

## Pushing the container to an internal registry

`podman push ilab-cli:latest docker://quay.io/bbrowning/ilab-cli:0.14.1-10`


## Deepspeed Container

## Building the container

`podman build -f containers/training/Containerfile -t ilab-deepspeed`

## Pushing the container

`podman push ilab-deepspeed:latest docker://quay.io/bbrowning/ilab-deepspeed:1.0.8`

