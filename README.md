# BeatMapCurveShapes
This repository contains a dataset of Beat Saber maps, scores, and players, originally sourced from BeatLeader, enhanced with multi-parameter absolutely monotonic PP curves that explain the scores well, and functionality to use this dataset.

The dataset is originally sourced from [BeatLeader](https://beatleader.com/), enriched with a 14-parameter parameterization of absolutely monotonic PP curve shapes for maps, that explain the scores well. The dataset is meant to be used by anybody attempting to produce map difficulty estimation algorithms from the map objects directly, as a supervised dataset of ground truth to target and/or evaluate with respect to. These parameterizations were produced with the code included in https://github.com/Undeceiver/BetaSaber, which by definition means every PP curve is absolutely monotonic, and aimed to find the PP curves for maps that were absolutely monotonic and best explained the scores.

_**This is not a suitable difficulty estimation algorithm for maps on its own because it depends on scores.**_

If you are planning to train your own difficulty estimation algorithm, you will need the map files themselves. These are way too much to include in the repository, but you can download them from Beat Saver using [BeatLeader's playlist](https://beatleader.com/playlist/ranked)

Apart from the code included in the repository, you can also see the results of these multi-parameter curves at https://portable.undeceiver.beatleader.pro/

It is likely I will write a more thorough academic publication on the process for obtaining this dataset within the next few months. I will link it here if/when that happens.

## Basic technical explanation

A function is _absolutely monotonic_ if all of its derivatives are positive. In more plain terms, the curve consistently curves upwards in every sense. This is ideal for PP curves because it means that the higher the score, the more that every bit of improvement is rewarded. The parameterization presented here is guaranteed by construction to be absolutely monotonic.

Moreover, this is a 14-parameter parameterization (though in practice 6-8 parameters would be sufficient for most maps). This contrasts with simple 1-3 parameter parameterizations in that it allows for **variable curve shapes**. In practical terms, this means that, for example, a high tech map can reward more for a bare pass, and grow more slowly for higher scores, while an acc map can give very little PP for just passing, but highly reward near 100% scores.

<img width="1124" height="453" alt="image" src="https://github.com/user-attachments/assets/4d150ddc-4470-43ae-9727-638ef90f06fc" />
<img width="1140" height="455" alt="image" src="https://github.com/user-attachments/assets/e677fcf0-e0e2-4845-917a-145a30b6ba20" />

These absolutely monotonic parameterizations are achieved by using an _exponential basis_:

We define a _unitary exponential_ to be the function $uexp_t(x) = {{e^{tx} - 1} \over {e^t - 1}}$. The reason we use unitary exponentials is that for any $t$, $uexp_t(0) = 0$ and $uexp_t(1) = 1$ ($1$ represents 100% score), so in some sense they are in the same scale.

We then represent absolutely monotonic functions as sums of unitary exponentials:

$$
pp(x) = \sum\limits_{i=1}^n w_i \cdot uexp_{t_i}(x)
$$

When $n = 7$, this then gives us a 14-parameter parameterization, where the 14 parameters are the 7 weights $w_i$ and the 7 exponents $t_i$. However, we do not represent the $t_i$ directly, because there would be a risk of them being too close and creating degenerate functions. Instead, we represent the $t_i$ by using a starting value of $1$, and representing how much each $t_{i+1}$ multiplies the previous $t_i$ (since multiplying $t_i$ by $k$ approximately reduces the space between the "inflection point" of that component to $1$ to $1/k$-th of what it was).

What this means is that our actual parameters are the 7 weights $w_i$ and 7 multipliers $\alpha_i$, so that $t_1 = \alpha_1$ and $t_{i+1} = \alpha_{i+1} \cdot t_i$. The $w_i$ and $\alpha_i$ are intercalated in the dataset: $[\alpha_1,w_1,\alpha_2,w_2,...,\alpha_7,w_7]$.

So, for example, if a map has parameters: $[2.087, 0.863, 2.132, 0, 2.132, 0, 2.132, 0, 2.132, 1, 1.314, 1, 1.104, 1]$, we have:

$$
\alpha_1 = 2.087,
w_1 = 0.863,
\alpha_2 = 2.132,
w_2 = 0,
\alpha_3 = 2.132,
w_3 = 0,
\alpha_4 = 2.132,
w_4 = 0,
\alpha_5 = 2.132,
w_5 = 1,
\alpha_6 = 1.314,
w_6 = 1,
\alpha_7 = 1.104,
w_7 = 1,
$$

and so

$$
t_1 = 2.087,
t_2 = 2.087 \cdot 2.132 = 4.45,
t_3 = 4.45 \cdot 2.132 = 9.486,
t_4 = 9.486 \cdot 2.132 = 20.225,
t_5 = 20.225 \cdot 2.132 = 43.119,
t_6 = 43.119 \cdot 1.314 = 56.658,
t_7 = 56.658 \cdot 1.104 = 62.551,
$$

and therefore the PP curve for this map is: 

$$
pp(x) = 0.863 uexp_{2.087}(x) + 0 uexp_{4.45}(x) + 0 uexp_{9.486}(x) + 0 uexp_{20.225}(x) + 1 uexp_{43.119}(x) + 1 uexp_{56.658}(x) + 1 uexp_{62.551}(x)
$$

or, simplified

$$
pp(x) = 0.863 uexp_{2.087}(x) + 1 uexp_{43.119}(x) + 1 uexp_{56.658}(x) + 1 uexp_{62.551}(x)
$$

The Python code included includes a simple function to do this calculation for you, but you can reproduce this in any other programming language.

## Contact

Please feel free to contact me on Discord (`undeceiver`) if you have questions or thoughts on this.
