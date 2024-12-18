# Preface

## What this book is about

Offers a framework for selecting AI related tools, but doesn't explain how to use those tools.

- Should I build this AI application?
- How do I evaluate my application? Can I use AI to evaluate AI outputs?
- What causes hallucinations? How do I detect and mitigate hallucinations?
- What are the best practices for prompt engineering?
- Why does RAG work? What are the strategies for doing RAG?
- What’s an agent? How do I build and evaluate an agent?
- When to finetune a model? When not to finetune a model?
- How much data do I need? How do I validate the quality of my data?
- How do I make my model faster, cheaper, and secure?
- How do I create a feedback loop to improve my application continually?

## Who this book is for

- Those who want to solve real world problems with foundation models

# Chapter 1. Introduction to Building AI Applications with Foundation Models

## The Rise of AI Engineering

Language models have been around for a while, but they've been able to scale to Large Language model, thanks to
`self-supervision`

### Language model

- predicting next token ...
- Model types
- Masked language model
    - trained to predict missing tokens anywhere in a sequence, using the context from both before and after the
      missing tokens
        - e.g. `My favorite ___ is blue`
    - e.g. BERT(bidirectional encoder representations from transformers)
    - commonly used for non-generational tasks such as sentiment and text classification
    - useful when the model has to understand the context
- Autoregressive language model
    - trained to predict the next token in a sequence using only the preceding tokens
        - e.g. `My favorite color is ___`
    - can continually generate one token after another
    - a model that can generate open-ended outputs is called `generative`, hence the term generative AI

### Self-supervision

- language models gained its popularity due to it's self-supervision
    - other ML models require supervision(aka labeling)
    - data labeling is expensive and time-consuming
- with self-supervision, instead of requiring explicit labels, the model can infer labels from the input data
    - cf) `self-supervision != unsupervision`, in self-supervision, labels are inferred from the input data. But
      in unsupervised learning, you don't need labeling at all

### From Large Language Models to Foundation Models 
