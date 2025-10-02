# Style Guide

## General
- Start headers with `##` cause the title assumes the `#` header

## Links
- Use `{{< relref "lec1/" >}}` for links to subdirectories
- Use `({{< relref "../../posts/ELiASA/chap6/6-1#arcsine-law" >}})` to link to more global stuff

### References section
[1] GitHub repo - nanoDiffGPT: https://github.com/hasithv/nanoDiffGPT

[2] Diffusion notes: [{{< ref "../FlowDiffusion/FlowDiff.md" >}}]({{< relref "../FlowDiffusion/FlowDiff.md" >}})

[3] LLaDa paper: https://arxiv.org/abs/2502.09992

[4] Block diffusion paper: https://arxiv.org/abs/2503.09573

[5] MMaDa paper: https://arxiv.org/abs/2505.15809


## Math Theorems
### Theorem
```
> **Theorem 10:** (Marginalization Trick) Let $u_t^\text{target}$ be a conditional vector field defined so that the ODE yields the conditional probability path $p_t(\cdot|z)$
$$
X_0 \sim p_{init}, \quad \frac{\text{d}}{\text{d}t} X_t = u_t^\text{target}(X_t) \implies X_t \sim p_t(\cdot|z).
$$
> Then, the marginal vector field $u_t^\text{target}$ given by
$$
u_t^\text{target}(x) = \int u_t^\text{target}(x|z) \frac{p_t(x|z) p_{data}(z)}{p_t(x)} \text{d}z
$$
> will yield the marginal probability path $p_t$:
$$
X_0 \sim p_{init}, \quad \frac{\text{d}}{\text{d}t} X_t = u_t^\text{target}(X_t) \implies X_t \sim p_t.
$$
> In other words, $u_t^\text{target}$ converts $p_{init}$ to $p_data$.
```

## Images
{{< figure src="./images/playrate.svg" width="400px" align="center" caption="">}}
