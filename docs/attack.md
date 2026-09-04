# The Attack: Shadow-Model Membership Inference

## The Question

Can you tell whether a specific person's data was used to train a model just by looking at how the model behaves?
If an attacker can determine that a person's record was in a model's training set, that alone can leak sensitive information, like confirming someone was part of a hospital dataset used to train a diagnosis-prediction model.
This is what's called a **membership inference attack**.

### Why are we even able to infer this at all?

It's because every model is trained to fit its training data.
When it fits the data too well, that's called **overfitting**, and that's somewhat like memorising specifics of the record instead of learning general patterns.

The practical effect is that when you feed a model data it was trained, it tends to respond with extremely confident probabilities.
When you feed it data that's similar but wasn't trained. the confidence score is generally lower even if it gets the answer right. 

## How the Attack Works

We're basing our attack on the 2017 paper **"Membership Inference Attacks against Machine Learning Models"** by Shokri et al.

The attacker doesn't have the target model's real training data.
If that was the case, there'd be nothing to infer.
The workaround is **shadow models**:

1. Train one or more shadow models on data from the same distribution as the target's (assumed) training data. You fully control these, so you know exactly what each was trained on.
2. Query each shadow model with its own training data ("in"/member) and with data it's never seen ("out"/non-member). Record the confidence scores both times.
3. This produces a labeled dataset: confidence scores -> member or not.
4. Train an **attack model** (a classifier) on this labeled data. It learns to recognize the confidence pattern that means "the real model was trained on this."
5. Point the trained attack model at the actual target model's outputs. It predicts membership for any query, without ever having seen the target's real training data.

**Why the attack reads confidence scores, not labels:** the signal lives in the shape of the probability distribution (how peaked or spread out it is), not in the final predicted class.
This is why the target model must expose `predict_proba(X)`, not just `predict(X)`.

## Some More Stuff

A single shadow model is enough for a working attack but its labeled dataset only reflects that one
model's specific quirks.
More shadow models, each trained on a different sub-sample, give the attack model more varied examples to generalize from, which generally improves success rate (with diminishing returns though).

**The plan right now:** start with 1 shadow model to validate the full pipeline end-to-end fastest
Once working, scale up to 5-10 and compare attack success rate.
This can also be a result in its own way.

Attack success rate should be meaningfully above 50% (random guessing).
If it isn't, check for split leakage or insufficient target-model overfitting before assuming the attack implementation itself is wrong.

## Implementation Notes

- There should be a 4-way data split, all disjoint.
  1. **Target train**: trains the target model
  2. **Target test**: evaluates target accuracy, untouched by the attack
  3. **Shadow train pool**: further split per shadow model; each shadow's own training subset = member examples
  4. **Shadow-out pool**: held out from all shadow training; non-member examples