# Pre-training, Fine-tuning, Inference, Parameters & Temperature

## 1. What is it?

- **Pre-training**: teaching the model language by predicting next tokens on massive text (internet, books). Very expensive, done once by the provider.
- **Fine-tuning**: extra training on a smaller, focused dataset to specialize the model (tone, format, domain).
- **Inference**: using the trained model to generate output. This is what happens every time YOU call the API.
- **Parameters**: the learned numbers inside the model (e.g., "7B model" = 7 billion parameters).
- **Temperature**: a runtime setting controlling randomness of token selection.

## 2. Why does it exist?

Pre-training gives general language ability. Fine-tuning adapts it cheaply (no need to retrain from scratch). Inference is the product you actually use. Temperature exists because sometimes you want deterministic answers (math, code) and sometimes creative ones (stories, marketing).

## 3. Simple Mental Model

```
Pre-training  = school education (general knowledge)
Fine-tuning   = job training (specific skills/style)
Inference     = actually doing the job
Parameters    = everything the model learned (frozen after training)
Temperature   = "creativity dial" used at inference time
```

## 4. How Temperature Works

The model gives a probability to every possible next token. Temperature reshapes the choice:

```
Low temperature (0.0 - 0.3)          High temperature (0.8 - 1.5)
- almost always picks top token      - picks from a wider spread
- predictable, consistent            - surprising, creative
- good: math, code, extraction       - good: stories, brainstorming
```

Pseudocode idea:

```
pick next token based on probabilities, adjusted by temperature
```

## 5. ASCII Architecture

```
TRAINING (once, by provider)              INFERENCE (every API call)
                                          
Massive text                              Your prompt
    |                                         |
    v                                         v
Pre-training  --> Base model              LLM forward pass
    |                                         |
    v                                         v
Fine-tuning   --> Tuned model             Token picked
    |                                     (temperature
    v                                      controls how)
Parameters FROZEN -----stored as weights----> used here
```

## 6. Important Terms

- **Weights**: another word for parameters.
- **Base model vs instruct model**: base = raw text predictor; instruct = fine-tuned to follow instructions (what you normally use).
- **Sampling**: the act of choosing the next token from probabilities.

## 7. What happens internally?

At inference: your tokens go through all layers → probability distribution → temperature adjusts it → one token sampled → appended → repeat.

## 8. How is it different?

| | Pre-training | Fine-tuning | Inference |
|---|---|---|---|
| When | Once | Occasionally | Every call |
| Cost | Millions of $ | Moderate | Per-token |
| Changes | Creates parameters | Adjusts parameters | Changes nothing |
| Who | Provider | Provider or you | You |

## 9. Common Misunderstandings

- ❌ "Temperature makes the model smarter/worse." It only changes randomness, not knowledge.
- ❌ "Fine-tuning teaches new facts reliably." It teaches style/format better than facts — that's a RAG job.
- ❌ "Inference trains the model." Normal API calls never change the model.

## 10. Connection to Previous

Previous: tokens and context — the units the model reads. This file: how the model got its ability (training) and how you steer each output (temperature).

## 11. Connection to Next

Even a perfectly trained model generates wrong-but-confident answers. That problem — hallucination — comes next.

## 12. One-Minute Explanation

"Pre-training teaches an LLM language by next-token prediction on huge text. Fine-tuning specializes it further on focused data. Every API call is inference — the parameters are frozen, nothing learns. Temperature is an inference-time dial: low makes output deterministic for code and math, high makes it creative. Temperature changes randomness, not correctness."

## Hinglish Explanation

Pre-training matlab model ko poora internet padha kar language sikhana. Fine-tuning matlab usko specific kaam ke liye extra train karna. Jab aap API call karte ho, wo inference hai — model kuch naya seekh nahi raha. Temperature ek "creativity knob" hai: kam temperature = same type ka jawab har baar, zyada temperature = creative but unpredictable jawab.
