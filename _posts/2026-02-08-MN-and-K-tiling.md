---
layout: post
title: "GEMM and Tilings"
date: 2026-02-08
tags: [CUDA]
---

# GEMM and Tilings

Tiling is a good concept and tool to abstract and measure the size of sub-tasks, which bridges the gap between data storage and task distribution. When using SIMT model to solve GEMM-like calculations, there are two perspectives to divide the problem. One is the MN-tiling and the other is the K-tiling (, given the computation is defined as $\begin{matrix}C = A * B\end{matrix}$, and $\begin{matrix}A \in R^{M, K}\end{matrix}$, $\begin{matrix}B \in R^{K, N}\end{matrix}$ and $\begin{matrix}C \in R^{M, N}\end{matrix}$)

## MN-tiling

This grid-like partition aims to __gather__ the __vector multiply__ result from the view of the output matrix $C$. The MN dimensions loop is an __all-to-all__ process, and thus it's easier to get high concurrency and feed the hardware by spliting tilings and paralleling along these axes.

However, constrained by the limited storage space in hardwares, this __all-to-all__ style computation will cause the repeated loading of inner matrix (e.g. the matrix $B$), which will lead to less __Arithmetic Intensity__ and maybe more crowded __memory instruction pipeline__ (MIO).

$$ \begin{equation}
Arithmetic\ Intensity \approx \frac{Total\ Computation} {Total\ Memory\ Load} = \frac{MNK} {MK + M*NK} = \frac{MN}{M + MN} 
\end{equation}
$$

## K-tiling

The native K-tiling sacrifices the concurrency (, not as many CTAs as MN-tiling), but it could reuse the loaded memory to save IO. And this makes it have stronger Arithmetic Intensity. Because of this feature, it's often used in the world of communication between SMEM and RF, and matmul, which is just the inner loop for a GEMM-like op.

$$ \begin{equation}
Arithmetic\ Intensity \approx \frac{Total\ Computation} {Total\ Memory\ Load} = \frac{MNK} {MK + NK} = \frac{MN}{M + N} 
\end{equation}
$$

But it could also be used to fit some __"tall and thin"__ matrices $A$ and $B$ (e.g. small $M$, $N$ and very large $K$) by launching CTAs with tilings along the $K$ dimmension. And of course, it needs an intermediate storage and final reduce kernel to sum all the $[M, N]$ shape sub-matrices.

## Final

As previously depicted, the different tiling could relieve different pressure. Considering the GEMM's computing pattern and the CTAs and warps level concurrency ability, for most balanced shape input $A$ and $B$, the GEMM often uses __MN-tiling__ to launch grids of CTAs to flood the SM, and in the SMEM $\Leftrightarrow $ RF level, __K-tiling__ could serve the scenario, as the __MN-tiling__ makes the remained $K$ dimension is relatively large, and it's fast and cheap to use register as intermediate storage. The full attention to prefill, e.g. flash attention, also follows this way.

But sometimes the $K$ dimmension is large enough, so that the task will be divided along K dimmension and then do __MN-tiling__ for inner loops. Also, it launch a reduction kernel on the same stream to merge the sub-matrices. The full attention to decode, e.g. paged attention, adopts this strategy.

Both parallelism styles have room to futher optimize, such as pre-fetch and double-buffer to better overlap the computation and memory IO, and for more complicated requirements and scenarios, specilizing the warp or persisting kernels.
