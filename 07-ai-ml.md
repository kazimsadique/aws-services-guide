# 07 — AI & Machine Learning

[🏠 Back to README](../README.md) · [🎯 Pick your level](../README.md#-pick-your-level) · [🌱 Start here](../README.md#-start-here-new-to-aws)

## Contents

| # | Service | Level |
|---|---------|-------|
| 7.1 | [Amazon Bedrock](#71-amazon-bedrock) | 🟢 Beginner |
| 7.2 | [Amazon SageMaker AI](#72-amazon-sagemaker-ai) | 🔴 Advanced |
| 7.3 | [Amazon Rekognition](#73-amazon-rekognition) | 🟢 Beginner |
| 7.4 | [Amazon Polly](#74-amazon-polly) | 🟢 Beginner |
| 7.5 | [Amazon Transcribe](#75-amazon-transcribe) | 🟢 Beginner |

| Need | Service |
|------|---------|
| Use foundation models (LLMs, image models) via API, build agents/RAG | **Bedrock** |
| Build, train, tune, deploy *your own* ML models | **SageMaker AI** |
| Ready-made image/video analysis | **Rekognition** |
| Text → speech | **Polly** |
| Speech → text | **Transcribe** |

---

## 7.1 Amazon Bedrock

> **Level:** 🟢 Beginner

**What it is:** Fully managed access to foundation models from multiple providers (e.g. Anthropic Claude, Amazon Nova, Meta Llama, Mistral, Cohere and others) through one API — your data isn't used to train the base models.

**Key concepts**
- **Converse API**: one unified chat API across models (with tool use / function calling and streaming)
- **Knowledge Bases**: managed RAG — ingest documents from S3 etc., chunk, embed and store in a vector store (OpenSearch Serverless, Aurora pgvector, S3 Vectors, etc.)
- **Agents** and **AgentCore**: build and run AI agents (runtime, memory, identity, gateway for tools, observability)
- **Guardrails**: content filters, denied topics, PII redaction, contextual grounding checks
- **Model customization**: fine-tuning, continued pre-training, distillation
- **Prompt management**, **Flows**, model evaluation
- Batch inference, provisioned throughput, cross-region inference profiles

**Python (boto3) — Converse API**
```python
import boto3

brt = boto3.client("bedrock-runtime", region_name="eu-north-1")
resp = brt.converse(
    modelId="<model-or-inference-profile-id>",
    messages=[{"role": "user", "content": [{"text": "Summarise what AWS Fargate is in one sentence."}]}],
    inferenceConfig={"maxTokens": 200, "temperature": 0.2},
)
print(resp["output"]["message"]["content"][0]["text"])
```

**Pricing:** on-demand per input/output token (varies by model), batch (discounted), or provisioned throughput; Knowledge Bases/Guardrails/AgentCore have their own charges.

**Gotchas:** Model availability varies by region — check before choosing a region; some models must be enabled in the account first.

[↑ Back to contents](#contents)

---

## 7.2 Amazon SageMaker AI

> **Level:** 🔴 Advanced

**What it is:** The end-to-end platform for building, training and deploying custom ML models. (The name "Amazon SageMaker" now also covers a broader unified data + AI platform — "SageMaker Unified Studio"; the classic ML service is called **SageMaker AI**.)

**Key concepts**
- **Studio** IDE / JupyterLab notebooks
- **Training jobs** (managed infra, Spot training, distributed training), **HyperPod** for large-scale foundation model training
- **Processing jobs**, **Data Wrangler**, **Feature Store**
- **JumpStart**: pre-trained and open models to fine-tune/deploy
- **Pipelines** (MLOps), **Model Registry**, **Experiments** (MLflow)
- **Inference**: real-time endpoints, serverless inference, asynchronous inference, batch transform
- **Clarify** (bias/explainability), **Model Monitor** (drift)

**Pricing:** per instance-hour for notebooks, training, processing and endpoints + storage.

**Gotcha:** Real-time endpoints run 24/7 — delete idle ones or use serverless/async inference.

[↑ Back to contents](#contents)

---

## 7.3 Amazon Rekognition

> **Level:** 🟢 Beginner

**What it is:** Pre-trained computer vision API for images and video.

**Capabilities:** object/scene labels, face detection and comparison, face search in collections, text in images, content moderation, celebrity recognition, PPE detection, **Custom Labels** (train on your own images), video analysis (stored or streaming).

**Python**
```python
import boto3

rek = boto3.client("rekognition")
labels = rek.detect_labels(
    Image={"S3Object": {"Bucket": "my-images", "Name": "photo.jpg"}}, MaxLabels=10
)["Labels"]
```

**Pricing:** per image or per minute of video processed; face metadata storage.

**Note:** Face-related features raise privacy and legal considerations (e.g. GDPR biometric data) — get legal review for EU use.

[↑ Back to contents](#contents)

---

## 7.4 Amazon Polly

> **Level:** 🟢 Beginner

**What it is:** Text-to-speech.

**Key concepts:** many voices and languages (including Swedish), voice engines (**standard**, **neural**, **long-form**, **generative**), **SSML** for pauses/emphasis/pronunciation, lexicons, speech marks (word timings for lip-sync/highlighting).

**Use it for:** accessibility, e-learning narration, IVR/voice bots, reading articles aloud.

**Pricing:** per million characters (higher for neural/generative).

**Python (boto3) — turn text into an MP3 with a Swedish voice**
```python
import boto3

polly = boto3.client("polly", region_name="eu-west-1")
resp = polly.synthesize_speech(
    Text="Hej! Välkommen till AWS.",
    OutputFormat="mp3",
    VoiceId="Elin",          # Swedish neural voice
    Engine="neural",
)
with open("welcome.mp3", "wb") as f:
    f.write(resp["AudioStream"].read())
```

[↑ Back to contents](#contents)

---

## 7.5 Amazon Transcribe

> **Level:** 🟢 Beginner

**What it is:** Automatic speech recognition (speech → text).

**Key concepts**
- Batch (files in S3) and **streaming** (real-time)
- Speaker diarization, channel identification, automatic language identification
- Custom vocabularies and custom language models
- PII redaction, toxicity detection
- **Transcribe Call Analytics** (sentiment, call summaries) and **AWS HealthScribe** (clinical notes)

**Use it for:** meeting/lecture transcripts, subtitles, call-center analytics, voice input for apps.

**Pricing:** per second of audio (minimum per request), tiered by volume.

**Typical pipeline:** Transcribe (audio → text) → Bedrock (summarise/extract) → Polly (answer → audio).

**Python (boto3) — transcribe an audio file stored in S3**
```python
import time
import boto3

tr = boto3.client("transcribe")
job = "meeting-2026-11-23"
tr.start_transcription_job(
    TranscriptionJobName=job,
    Media={"MediaFileUri": "s3://my-audio/meeting.mp3"},
    MediaFormat="mp3",
    IdentifyLanguage=True,            # or LanguageCode="sv-SE" / "en-US"
    OutputBucketName="my-transcripts",
)

while True:
    status = tr.get_transcription_job(TranscriptionJobName=job)["TranscriptionJob"]
    if status["TranscriptionJobStatus"] in ("COMPLETED", "FAILED"):
        break
    time.sleep(10)
print(status["TranscriptionJobStatus"])  # transcript JSON is now in the output bucket
```

[↑ Back to contents](#contents)

---

[← 06 Databases](06-databases.md) · [🏠 README](../README.md) · [08 Application Integration (Coordination) →](08-application-integration.md)
