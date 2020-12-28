---
layout: post
title: Notes on "Neural Scene De-Rendering"
---

["Neural Scene De-rendering"](http://nsd.csail.mit.edu/), Wu, Tenenbaum, Kohli. CVPR 2017.

The authors develop a new interpretable scene representation. It's an encoder-decoder architecture, but the decoder is a deterministic non-differentiable renderer. The encoder has two pieces: a proposal generator and an object interpreter. The proposal generator uses [MNC](https://arxiv.org/abs/1512.04412), a segmentation method which predicts masks of candidate objects. The proposal generator computes 100 proposals per image. The object interpreter predicts, for each proposal, whether there is an object in the segment.

The internal representation (the code) is "Scene XML", which explicitly represents object category, size, color, position and pose. The objects are stereotyped shapes. Because the objects have depth, occlusion is possible, so two Scene XML codes with small edit distances can generate very different images. During training, the authors want to minimize both Scene XML distance and render distance. The renderer is non-differentiable, so the authors use a stochastic layer at the end of the encoder and REINFORCE. They can also tune just the render loss without scene XML labels, although this only works in a mixed curriculum-learning setting.

The authors also consider "analysis-by-synthesis refinement", where they treat the position as an initialization for a Gibbs sampler, run updates, and see whether it renders better. (I think this is basically just random search around the initial proposal in representation space.)

The authors use two experimental settings, the [Abstract Scene Dataset](https://vision.ece.vt.edu/clipart/) and a new Minecraft dataset with 12 different objects. The methods work better than some CNN or CNN+LSTM baselines, although maybe not quite as convincingly better as I might have hoped. Numerically the analysis-by-synthesis refinement helps, but in the examples they show I don't see any differences. Interestingly, on the Minecraft dataset, the reconstruction has 4x lower error than the CNN+LSTM baseline (5.05 vs 20.22), but humans only prefer the NSD images at < 60%. The authors mention that "margins are smaller on the Minecraft dataset because all algorithms perform better", but the tables don't fully support this, and I don't have any intuition for why the Minecraft task is easier. 

Section 5 contains some fun qualtiative applications. Image editing is simple in this representation: just modify the Scene XML and you get what you expect. Inpainting if you can still see part of the object works fine (it shows up in the Scene XML); there's no real study of whether the network is learning semantics of related objects, although I'd guess it isn't? They look at image captioning. My favorite example is the visual analogies, although it's more cute than substantial. Given two images $$A$$ and $$A'$$, the authors construct representations $$Z_A$$ and $$Z_{A'}$$ and do a depth first search to find the minimum edit distance in representation space. The authors apply this edit to $$Z_B$$ to generate a $$Z_{B'}$$, completing the analogy. It's cute, but it only works because the authors restrict to the operations of changing the pose, position, or category of an object, duplicating or removing an object, or swapping two objects: we're not going to solve [Bongard problems](https://en.wikipedia.org/wiki/Bongard_problems) this way.

Code for the model and experiments don't seem to be available.

#mlpapers
