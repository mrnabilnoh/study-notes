# AWS AI Practitioner (AIF-C01) Study Notes

This is a condensed study note for the AWS Certified AI Practitioner (AIF-C01) exam. It focuses on the core ideas, AWS services, and domain knowledge you need to understand and review.

For a deeper write-up and more detailed notes, see: https://www.nabilnoh.com/posts/aws-ai-practitioner-aif-c01-study-notes/

## Exam overview

| Domain | Topic | Weight |
| :--- | :--- | :---: |
| Domain 1 | Fundamentals of AI and Machine Learning | 20% |
| Domain 2 | Fundamentals of Generative AI | 24% |
| Domain 3 | Applications of Foundation Models | 28% |
| Domain 4 | Guidelines for Responsible AI | 14% |
| Domain 5 | Security, Compliance, and Governance for AI Systems | 14% |

## Domain 1: Fundamentals of AI and ML

### Core learning types

- Supervised learning
  - Classification: predicts a category
  - Regression: predicts a numeric value
- Unsupervised learning
  - Clustering: groups similar data points together
  - Dimensionality reduction: reduces feature count while preserving important information
- Reinforcement learning
  - Agent learns by interacting with an environment and receiving rewards

### Common model problems

| Concept | Meaning | Typical fix |
| :--- | :--- | :--- |
| Underfitting | Model is too simple and misses patterns | Increase complexity, add features |
| Overfitting | Model learns noise and fails to generalize | Add regularization, use early stopping, gather more data |

### Model evaluation metrics

- Precision: useful when false positives are costly
- Recall: useful when false negatives are costly
- F1 score: balances precision and recall
- MAE and RMSE: used for regression performance

### Key idea

For AI exams, it helps to know not just the formula but also the business meaning behind the metric. A healthcare model may prioritize recall, while a spam filter may prioritize precision.

## Domain 2: Fundamentals of Generative AI

### Transformer basics

- Tokens are the basic units processed by LLMs.
- Embeddings turn tokens into vectors that capture meaning.
- Self-attention lets the model weigh the importance of different words in a sequence.
- Decoder-only models are commonly used for text generation.
- Encoder-only models are strong for understanding and embedding tasks.
- Encoder-decoder models are used for tasks such as translation and summarization.

### Model behavior controls

| Parameter | Purpose |
| :--- | :--- |
| Temperature | Controls randomness in output |
| Top-p | Limits sampling to the most likely tokens within a probability threshold |
| Top-k | Limits sampling to the most likely k tokens |
| Max tokens | Caps response length |

### Practical reminder

- Lower temperature usually gives more deterministic output.
- Higher temperature gives more creative or varied output.
- Max tokens prevents runaway or very long responses.

## Domain 3: Applications of Foundation Models

### Model adaptation options

- Prompt engineering: quick and low cost
- RAG: retrieves current knowledge and adds it to the context
- Fine-tuning: adapts the model to a specific domain or tone
- Continued pre-training: deeper domain adaptation with more compute and data

### AWS AI services worth knowing

- Amazon Bedrock
  - Managed access to foundation models
  - Supports model selection through a common API
- Knowledge Bases for Amazon Bedrock
  - Good fit for RAG workflows
  - Connects to vector stores and internal data sources
- Agents for Amazon Bedrock
  - Helps orchestrate multi-step actions and tool use
- Guardrails for Amazon Bedrock
  - Adds filters for content safety, PII, and topic restrictions
- Amazon Q
  - AI assistant for business and developer workflows
- Amazon SageMaker JumpStart
  - Quick access to popular models and solutions

### When to use what

- Prompt engineering: for small, fast tasks
- RAG: when you need current and traceable business knowledge
- Fine-tuning: when the model needs a more specific style or domain behavior
- Bedrock agents: when the workflow needs tool use and orchestration

## Domain 4: Responsible AI

Responsible AI focuses on building systems that are fair, understandable, safe, and privacy-aware.

### Core pillars

- Fairness: reduce bias in data and model behavior
- Explainability: help humans understand model decisions
- Robustness: protect against failure and adversarial misuse
- Privacy: handle sensitive data carefully
- Transparency: clearly communicate how AI is being used

### AWS tools

- Amazon SageMaker Clarify
  - Helps detect bias and explain feature influence
- Amazon Bedrock Guardrails
  - Filters harmful content and redacts sensitive data
- Amazon A2I
  - Adds human review for low-confidence predictions
- AWS AI Service Cards
  - Provides safety, limitations, and intended use guidance

## Domain 5: Security, Compliance, and Governance for AI Systems

### Shared responsibility model

AWS manages the underlying infrastructure and model hosting environment. Customers remain responsible for:

- their data
- prompts and outputs
- IAM permissions
- security policies
- fine-tuning data and governance controls

### Security controls to know

- Encryption in transit and at rest
- AWS KMS for data encryption keys
- VPC access and AWS PrivateLink for private connectivity
- CloudTrail for API auditing
- CloudWatch for monitoring and logs
- SageMaker Model Monitor for drift detection
- AWS Artifact for compliance documentation

### Important AWS Bedrock privacy point

Customer data used in Amazon Bedrock is not used to train base AWS foundation models. That is an important exam concept and a key trust point for enterprise usage.

## Quick revision checklist

Before the exam, make sure you can explain:

- difference between supervised, unsupervised, and reinforcement learning
- underfitting vs overfitting
- when to use precision vs recall
- how Transformer models use self-attention and embeddings
- what temperature, top-p, top-k, and max tokens do
- when to choose prompt engineering, RAG, or fine-tuning
- what Amazon Bedrock, Bedrock Knowledge Bases, and Bedrock Agents do
- why responsible AI matters
- how to secure AI workloads and monitor them with AWS tools

## Final reminder

This exam is about understanding AI concepts in context, especially within AWS services and enterprise use cases. Focus on the practical meaning of each feature, not only the exact wording.
