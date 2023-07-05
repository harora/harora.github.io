---
layout: post
title: Notes on "Image Transformer"
---

["Image Transformer"](https://arxiv.org/abs/1802.05751), Parmar, Vaswani, Uszkoreit, Kaiser, Shazeer, Ku, Tran. ICML 2018.

"Image Transformer" presents a fully-visible (no latent variable) model for image generation. The authors apply it to unconditional CIFAR-10 and ImageNet images, and conditional CIFAR-10 generation. The authors also explore super-resolution on CIFAR-10 and CelebA.

The central idea in this paper is <em>self-attention</em> applied to image generation. Self-attention was also the core of the ["Attention Is All You Need"](https://arxiv.org/abs/1706.03762) and ["Tensor2tensor for neural machine translation"](https://arxiv.org/abs/1803.07416) papers, largely by the same authors. I read the "Attention Is All You Need" paper when it first came out and didn't fully understand it, and haven't read the "Tensor2Tensor for NMT" paper, so I had a lot to learn. In the intervening months, self-attentions seems to have become comprehensible to me, although this paper had a lot of subtlety given my current level of knowledge, and I'm left with quite a few questions.

The basic idea of a self-attention layer is that we have an input representation of the desired size, $$q$$, and we want to produce a new representation $$q'$$ of the same size. Stack up $$d$$-dimensional representations of all $$n$$-points (including the current point) into an $$n$$-by-$$d$$ matrix $$M$$. The crucial equation (Eq. 1 of the paper, simplified) is:

$$q' = {\tt softmax}\left((W_q q)(M W_k)^T)\right) MW_v$$

where the $$W$$ are all learned weight matrices. In words, $$W_q q$$ is a query in attention space. $$M W_K$$ is your database of $$n$$ responses in query space, their product is an $n$-vector describing the salience of each of the $$n$$ points, and the $$MW_v$$ term takes you back to query (or attention) space. There's dropout and layernorm and residual connections and [those are certainly important in ways I don't fully understand yet](https://arxiv.org/abs/1804.00247), but that's the basic idea.

It's crucial to keep in mind that even though we're going to stack these things in layers (we might compute ~10 different intermediate image-sized representations), because we want a tractable likelihood at the end, the *entire* computation for position $$i$$ cannot depend in any way on positions $$> i$$, so careful masking is involved. I admit that I'm still unclear how the *first* image-sized representation is generated, which seems like a related issue.

For parallelization, the basic scheme is that we divide the image into non-overlapping *"query blocks"*, each query block is embedded in a larger *"memory block"* (different memory blocks can overlap). Within a query block, each position attends to all positions in the associated memory block, and computation can be parallelized both within and across query blocks (I think). 

It's worth knowing a bit about other image generation models. The previous state-of-the-art for (non-GAN) models is from other "tractable likelihood" models that model the joint distribution of pixels in the image as a product of conditional distributions. In [the original PixelRNN model](https://arxiv.org/abs/1601.06759), each pixel depended on all previous pixels, but PixelCNN variants with bounded receptive fields lead to computationally simpler parallelization during training and are now far more popular. For a pixel CNN, the number of parameters scales with the receptive field size. The authors point out that for self-attention, the number of parameters is independent of the receptive field size (in the equation above ,the size of the receptive field $$n$$ shows up only in the sizes of computed quantities, not in any of the weight matrices $$W$$). On the other hand, the computation time certainly scales with the receptive field size, so it's unclear to me why this is so important.

There are a bunch of details of network architecture that I won't sum up here. The authors mention that the code is available at the [Tensor2Tensor github repository](https://github.com/tensorflow/tensor2tensor); the repo certainly includes many mentions of the Image Transformer model, so it should be excellent for diving deeper, although I did not immediately see evidence of "here's how to reproduce the models and experiments in the paper."

For image generation, the authors claim they "significantly improve the SOTA on ImageNet", going from the best previously published 3.83 to 3.77 NLL (all numbers in bits/dimension). The paper seems not to include any generated ImageNet images, so it's hard to evaluate this claim. (The authors also refer to the at-the-time-unpublished [PixelSNAIL](https://arxiv.org/abs/1712.09763), which from the abstract <em>also</em> uses self-attention, and gets NLL 3.8 on ImageNet and 2.85 to ImageTransformer's 2.9 on CIFAR-10). They show conditionally generated images for CIFAR-10; they look plausibleish, but I certainly can't immediately tell the difference between a 2.99 bits/dim model and a 3.03 bits/dim model. 

For super-resolution, they consider upsampling images from 8x8 to 32x32. On this task, they are able to make images that fool humans in a paired choice test about a third of the time, a big increase from referenced previous work which could only do so about 10% of the time.

The authors seem primarily focussed on achieving low NLL's. They mention when discussing super resolution that "automated metrics like pSNR, SSIM and MS-SSIM have been shown not to correlate with perceptual image quality." They also mention that the class-conditioned Image Transformer on CIFAR-10 achieves similar NLL as unconditioned, but with significantly higher perceptual quality (I assume measured by "eyeball"). This leaves me wondering why we care about optimizing NLL in the first place. In that sense, I think the human eval on super-resolution is the most interesting metric in the paper.

I've elided many details about checkpoint averaging, training procedures, 1d vs 2d local attention fields, and a zillion other things. Overall, I think self-attention is potentially a quite useful idea. I'm not super convinced by the experimental results: I would've wanted to see more "obviously better pictures" in something like this or understand what we should care about when evaluating these models better or ideally both, but it's fine. I also would've preferred more in-depth analysis of the computation costs and a deeper comparison to PixelCNNs.

Questions:
- It is true that for self-attention, the number of parameters is independent of the receptive field size. However, the computational cost still grows linearly with the size of the receptive field. If we're reducing number of parameters but not computation, what's the big win? Since this is a central claim of the paper, I'd love to see more detail here.
- Does the github repo make it easy to reproduce the results in the paper, given access to enough GPUs?
- The authors often mention one model is "significantly" better than another, but how is "significanty" measured? For instance, on CIFAR-10, the authors state that PixelSNAIL's perplexity of 2.85 bits/dim is significantly lower than Image Transformer's 2.99, but on ImageNet, Image Transformer's 3.77 is significantly lower than PixelSNAIL's 3.80. How are we evaluating significance? When or why should I care?
- How should I think about evaluation of these models more generally?
- The ImageNet result of 3.77 bits/dim uses checkp averaging. How much of the win came from model checkpointing? Were previous competing models checkpoint averaged, and if not, how much could they be improved? 
- In either application, how is the *first* image-sized representation generated?
