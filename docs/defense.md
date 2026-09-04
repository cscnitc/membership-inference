# The Defense: Differentially-Private Stochastic Gradient Learning

We're basing our defense on the 2016 paper **"Deep Learning with Differential Privacy"** by Abadi et al.

The model here leaks data because it comes from overfitting, from a model memorizing individual training examples too deeply.
The defense strategy is clear then.
We make it mathematically impossible for any single training example to influence the model too much.

The way to do this is through Differentially Private Stochastic Gradient Descent, or **DP-SGD**.

## How does this work?

Normal training computes a gradient from a batch of examples (the direction to adjust the model's weights) and updates the model in that direction.
DP-SGD adds two changes to this process:
1. **Per-sample gradient clipping**
    - Instead of one gradient for the whole batch, compute a separate gradient for each individual example.
    - Then clip each one to a fixed maximum norm.
    - This guarantees no single example can produce an outsized update on its own, mo matter how weird it is.
2. **Calibrated noise injection**
    - After you've clipped it, add random Gaussian noise to the sum of the now bounded per-example gradients before applying the update.
    - This noise is what gives the privacy guarantee, making it harder to perform the membership inference attack.

There's also something called the privacy budget, denoted by *epsilon*.
You can tune this parameter, with smaller epsilon being more noise and a stronger privacy guarantee, but also makes training harder and disrupts it, costing accuracy.
Larger epsilon is less noise, a weaker privacy guarantee, but closer to normal training which makes accuracy higher.
It's a slider of sorts and we have to find the perfect position.

One more note: there's a PyTorch library called Opacus that will do all of this automatically, these two steps.
Just calling the library doesn't help us understand anything though, so we'll be implementing this manually.
We can use Opacus at the end to see if our work gives similar results to an actually working implementation at the same epsilon level.

## Implementation Notes

- Both target architectures (Model A, Model B) must avoid DP-SGD-incompatible layers (no BatchNorm; use LayerNorm/GroupNorm or skip normalization).