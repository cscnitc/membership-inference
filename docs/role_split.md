# Role Split

4 people: 3 hands-on and a lead.

## Lead

Mostly coordinating everything and the final evaluation part.
Help out with writing as well.
Evaluation criteria could be attack accuracy, precision and recall, ROC-AUC, etc.

## Trainer

- Responsible for training both models and the data pipeline.
- Load and preprocess the [UCI diabetes dataset](https://archive.ics.uci.edu/dataset/296/diabetes+130-us+hospitals+for+years+1999-2008).
- Create a 4-way split: target test, target train, shadow train, shadow out
- Make sure to assert its disjoint, any overlap invalidates the entire project
- Build and train Model A, pretty simple one cause we need overfitting-prone models to make the attack easy.
- Also provide an interface to expose the probabilities
- After this, do the same thing with Model B, a larger or deeper architecture

## Attacker

- Train N shadow models on the shadow train pool, each getting its own split into this specific shadow's training data vs held-out
- Choose N, more gives better results but its diminishing, choose in the range of 5 to 20 initially. Make this a configurable parameter. 
- Query each shadow in training and held out, record confidence score and build a labeled dataset
- Train attack classifier or this dataset, one attack sub-model per output class according to the paper. (confidence patterns can change based on the class)
- Attack the target model from training guy and record attack accuracy.
- Once Model B is ready, do the same thing for that.

## Defender

- Build and train copies of Model A and B, match the exact same architecture as training guy.
- Build your own data loading and a normal train/test split, no need to do the 4 way split like the attack
- Implement DP-SGD manually with gradient clipping and Gaussian noise
- Get defended Model A working on a single epsilon value, confirm it trains normally (as in accuracy is a bit lower, not completely broken), and then sweep across a range of epsilon values
- Repeat on Model B
- Maybe implement with Opacus as well to check against these results. It's a very simple addition, like 2 or 3 lines of code.

## Extra notes

There's an extra tiny research question we can do here.
This is the reason for training 2 models as well instead of one.
Does a larger model, one more prone to overfitting, leak more data under this attack compared to a smaller one?
And does defending the larger model cost more accuracy to reach the same level of protection?
We'll answer these automatically as we construct the plot.
