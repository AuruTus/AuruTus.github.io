---
layout: post
title: "GEMM and Tilings"
date: 2026-02-08
tags: [CUDA]
---

# GEMM and Tilings

Tiling is a good concept and tool to abstract and measure the size of sub-tasks, which bridges the gap between data storage and task distribution. When using SIMT model to solve GEMM-like calculations, there are two perspectives to divide the problem. One is the MN-tiling and the other is the K-tiling (, given the computation is defined as $\begin{matrix}C = A * B\end{matrix}$, and $\begin{matrix}A \in R^{M, K}\end{matrix}$, $\begin{matrix}B \in R^{K, N}\end{matrix}$ and $\begin{matrix}C \in R^{M, N}\end{matrix}$)

## MN-tiling

This grid-like partition aims to __gather__ the __vector multiply__ result from the view of the output matrix $C$. The MN dimensions loop is an __all-to-all__ process, and thus it's easier to get high concurrency and feed the hardware by splitting tilings and parallelizing along these axes.

However, constrained by the limited storage space in hardware and the design of SIMT model, this __all-to-all__ style computation will cause the repeated loading of inner matrix (e.g. the matrix $B$), which will lead to less __Arithmetic Intensity__ and maybe more crowded __memory instruction pipeline__ (MIO).

$$ \begin{equation}
\begin{aligned}
Arithmetic\ Intensity &\approx \frac{Total\ Computation} {Total\ Memory\ Load} \\
&= \frac{B_M B_N K \cdot T_M T_N} {(B_MK + KB_N) \cdot T_M T_N} \\
&= \frac{MNK}{M K T_N + K N T_M} \\
&= \frac{MN}{M T_N + N T_M}
\end{aligned}
\end{equation}
$$

*where $[B_M, B_N]$ are the block (tile) sizes, $T_M = M / B_M$ and $T_N = N / B_N$ are the number of tiles along each axis.*


## K-tiling

The native K-tiling sacrifices the concurrency (, not as many CTAs as MN-tiling), but it could reuse the loaded memory to save IO. And this makes it have stronger Arithmetic Intensity. Because of this feature, it's the natural choice for the SMEM-to-RF inner loop, where loaded data is reused across the K-reduction within a single CTA.

$$ \begin{equation}
\begin{aligned}
Arithmetic\ Intensity &\approx \frac{Total\ Computation} {Total\ Memory\ Load} \\
&= \frac{M N B_K \cdot T_K} {(M B_K + B_K N) \cdot T_K} \\
&= \frac{MNK} {MK + NK}\\
&= \frac{MN}{M + N} 
\end{aligned}
\end{equation}
$$

*where $B_K$ is the block (tile) size, $T_K = K / B_K$ is the number of tiles along the K axis.*

But it could also be used to fit some __reduction-heavy__ matrices $A$ and $B$ (e.g. small $M$, $N$ and very large $K$) by launching CTAs with tilings along the $K$ dimension. And of course, it needs an intermediate storage and final reduce kernel to sum all the $[M, N]$ shape sub-matrices.

And it's easy to see that when $T_N = 1$ and $T_M = 1$, __equation__ $(1)$ reduces to __equation__ $(2)$. That's just the ideal Arithmetic Intensity for a GEMM. In fact, equation $(1)$ simplifies to $B_M B_N / (B_M + B_N)$ — the arithmetic intensity depends only on tile shape, making tile size the key tuning knob for the compute-vs-memory tradeoff.


## Final

As previously depicted, the different tiling could relieve different pressure. Considering the GEMM's computing pattern and the CTAs and warps level concurrency ability, for most balanced shape input $A$ and $B$, the GEMM often uses __MN-tiling__ to launch grids of CTAs to flood the SM, and in the SMEM $\Leftrightarrow $ RF level, __K-tiling__ could serve the scenario, as __MN-tiling__ leaves the K dimension relatively large, and it's fast and cheap to use register as intermediate storage. Prefill-phase attention (e.g., FlashAttention) also follows this way.

But sometimes the $K$ dimension is large enough, so that the task will be divided along K dimension and then do __MN-tiling__ for inner loops. Also, it launches a reduction kernel on the same stream to merge the sub-matrices. Decode-phase attention (e.g., PagedAttention) adopts this strategy.

Both parallelism styles have room to further optimize, such as pre-fetch and double-buffer to better overlap the computation and memory IO, and for more complicated requirements and scenarios, specializing the warp or persisting kernels.
