# Membership Inference Attack and Privacy Defense

A reproduction of a membership inference attack and a defense against it with DP-SGD.

## What are we doing?

Machine learning models can accidentally leak whether a given data point was part of their training set, and we're reproducing the shadow-model attack that exploits this (Shokri et al., 2017).
We'll then defend against it using DP-SGD (Abadi et al., 2016), which adds noise during training to hide that signal.
Finally, we'll test both across two model sizes and different privacy levels, to see how much accuracy the defense actually costs.

## Stuff to read

1. [Attack](docs/attack.md)
2. [Defense](docs/defense.md)
3. [Role Split](docs/role_split.md)
