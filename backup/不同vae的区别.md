### 第一类：基础与正则化 (Foundation \& Regularization)

*这一类模型主要修改 Loss 函数中的正则项，试图在重建质量和生成能力之间寻找更好的平衡。*

#### 1. **VAE (Standard Variational Autoencoder)**

* **核心**: 经典的 ELBO = 重建误差 + KL 散度（拉向标准正态分布）。
* **关键代码**:

```python
def loss_function(recon_x, x, mu, log_var):
    recon_loss = F.mse_loss(recon_x, x, reduction='sum')
    # 经典高斯 KL 公式：-0.5 * sum(1 + log_var - mu^2 - exp(log_var))
    KLD = -0.5 * torch.sum(1 + log_var - mu.pow(2) - log_var.exp())
    return recon_loss + KLD
```


#### 2. **Beta-VAE**

* **核心**: 在 KL 项前加系数 $\beta$。$\beta > 1$ 强迫解耦，$\beta < 1$ 提升重建。
* **关键代码**:

```python
# 唯一的区别就是乘了一个 beta
return recon_loss + self.beta * KLD
```


#### 3. **RAE_L2 (Regularized AE with L2)**

* **核心**: 放弃随机性（不像 VAE 那样采样），直接对 Latent Code 或解码器权重施加 L2 正则，防止过拟合。它更像 AE。
* **关键代码**:

```python
z = encoder(x) # 确定性输出，没有 mu, log_var
recon_loss = F.mse_loss(decoder(z), x)
# 对 Latent z 进行 L2 正则，限制其大小
reg_loss = 0.5 * torch.sum(z ** 2) 
return recon_loss + self.reg_weight * reg_loss
```


#### 4. **RAE_GP (Regularized AE with Gradient Penalty)**

* **核心**: 对解码器的输出关于潜在变量 $z$ 的梯度施加惩罚，强迫潜在空间平滑（类似 WGAN-GP）。
* **关键代码**:

```python
z.requires_grad_(True)
recon = decoder(z)
# 计算 Decoder 输出对 z 的梯度
grad_outputs = torch.ones_like(recon)
gradients = torch.autograd.grad(
    outputs=recon, inputs=z, grad_outputs=grad_outputs, create_graph=True
)[0]
# 梯度惩罚：希望梯度范数接近 0 (平滑)
gp_loss = (gradients.norm(2, dim=1) ** 2).mean()
return recon_loss + self.reg_weight * gp_loss
```

***

### 第二类：解耦学习 (Disentanglement)

*这一类模型的目标是让 Latent 的每一维代表一个独立的语义（如颜色、形状）。它们通过修改 KL 散度的计算方式来实现。*

#### 5. **Beta-TCVAE (Total Correlation VAE)**

* **核心**: 把 KL 拆解，单独惩罚“总相关性 (Total Correlation, TC)”。TC 是衡量变量间独立性的指标。
* **关键代码**:

```python
# 需要通过 mini-batch 估计 q(z) 的密度
# 这是一个复杂的估计过程，核心是计算 log_q_z 和 log_prod_q_z (假设独立时的密度)
log_q_z = log_density_gaussian(z, mu, log_var) # 联合分布
log_prod_q_z = log_density_gaussian(z, mu, log_var).sum(1) # 边缘分布乘积

tc_loss = (log_q_z - log_prod_q_z).mean() # TC = 联合 - 边缘乘积

return recon_loss + alpha * mi_loss + beta * tc_loss + gamma * dw_kld
```


#### 6. **FactorVAE**

* **核心**: 引入一个**判别器 (Discriminator)** 来区分“真实的 $z$”和“打乱维度后的 $z$”。如果判别器分不出来，说明各维度是独立的。
* **关键代码**:

```python
# 1. 真实的 z
z = encoder(x).embedding
# 2. 随机打乱维度的 z (Permuted)
z_permuted = permute_dimensions(z)

# 3. 判别器 loss
d_z = D(z)
d_z_perm = D(z_permuted)
# VAE 希望骗过判别器 (让 z 和 z_permuted 看起来一样)
tc_loss = (d_z[:, 0] - d_z[:, 1]).mean()

return recon_loss + KLD + self.gamma * tc_loss
```

***

### 第三类：提升后验与先验 (Expressive Posteriors \& Priors)

*标准 VAE 假设先验和后验都是高斯分布，这太简单了。这一类模型引入更复杂的分布。*

#### 7. **VAE_IAF (Inverse Autoregressive Flow)**

* **核心**: 编码器输出一个简单的 $z_0$，然后通过一系列可逆流（Flows）变换，变成复杂的 $z_k$。
* **关键代码**:

```python
# 1. 基础高斯采样
z0 = reparameterize(mu, log_var)

# 2. 通过 IAF Flow 变换 (通常是多层)
# 每一层根据之前的维度更新当前维度
for flow in self.iaf_flows:
    z0 = flow(z0) # z 变得不再是高斯分布了

return decoder(z0) # 用变换后的 z 重建
```


#### 8. **VAE_LinNF (Linear Normalizing Flow)**

* **核心**: 类似 IAF，但变换是线性的（Linear Flows），计算更便宜，能处理变量间的协方差。
* **关键代码**:

```python
# 对 z 进行一系列可逆线性变换
# 核心在于计算 Jacobian 行列式的 log
z, log_det = self.linear_flows(z0)
# Loss 里要加上这个 log_det
return recon_loss + KLD - log_det 
```


#### 9. **VAMP (Variational Mixture of Posteriors)**

* **核心**: 既然标准高斯先验不好，那我就**学习**一个先验。先验被定义为 K 个“伪输入”经过编码器后的混合分布。
* **关键代码**:

```python
# 伪输入 (Trainable Parameter)
u = self.pseudo_inputs 
# 先验的参数由伪输入通过编码器得到
prior_mu, prior_log_var = encoder(u)

# 计算 z 与这个 "混合高斯先验" 的 KL
# 公式比标准 KL 复杂，涉及 LogSumExp
log_p_z = log_normal_mixture(z, prior_mu, prior_log_var) 
log_q_z = log_normal(z, mu, log_var)

return recon_loss + (log_q_z - log_p_z).mean()
```

***

### 第四类：几何与离散潜在空间 (Geometric \& Discrete)

*改变 Latent Space 的物理性质。*

#### 10. **S-VAE (Spherical VAE)**

* **核心**: 潜在空间是**超球体表面**，而不是平面。分布使用 von Mises-Fisher (vMF)。
* **关键代码**:

```python
# 使用 vMF 分布采样 (方向向量)
dist = VonMisesFisher(mu, concentration) 
z = dist.rsample()

# KL 散度计算 (vMF vs 均匀超球分布)
# 涉及贝塞尔函数 (Bessel functions)
KLD = dist.kl_divergence(HypersphericalUniform(dim))
```


#### 11. **PoincareVAE**

* **核心**: 潜在空间是**双曲空间 (Hyperbolic Space)**，适合表示树状层级结构数据。
* **关键代码**:

```python
# 黎曼几何操作：将欧氏空间的 z 投影到庞加莱球上
z_hyp = expmap(z_eucl) 
# 距离度量不再是 MSE，而是双曲距离
dist = acosh(1 + 2 * ... )
```


#### 12. **VQ-VAE (Vector Quantized)**

* **核心**: 离散化。Latent 不是连续向量，而是 Codebook 中的索引。
* **关键代码**:

```python
# 1. 找最近邻 (Nearest Neighbor)
distances = (z - codebook).pow(2).sum()
indices = torch.argmin(distances, dim=1)

# 2. 量化
z_q = codebook[indices]

# 3. 梯度直通 (Straight Through)
z_q = z + (z_q - z).detach() # 前向用 z_q, 反向梯度给 z

# Loss: 重建 + Embedding Loss + Commitment Loss
loss = recon + ||sg[z]-e|| + beta * ||z-sg[e]||
```

***

### 第五类：对抗与感知 (Adversarial \& Perceptual)

*引入 GAN 的思想或感知 Loss 来提升清晰度。*

#### 13. **AAE (Adversarial AE)**

* **核心**: 用 GAN 的判别器来替代 KL 散度。判别器判断 z 是来自 Encoder 还是来自 Standard Normal。
* **关键代码**:

```python
# Encoder 阶段
d_fake = D(encoder(x))
# Generator Loss: 骗过判别器
g_loss = -torch.mean(torch.log(d_fake))

# Discriminator 阶段
z_real = torch.randn_like(z)
d_real = D(z_real)
d_loss = -torch.mean(torch.log(d_real) + torch.log(1 - d_fake))
```


#### 14. **VAEGAN**

* **核心**: 解码器的 Loss 不再是 MSE，而是判别器提取的特征层之间的误差（Perceptual Loss）。
* **关键代码**:

```python
# 重建图像
x_recon = decoder(z)

# 把真图和假图都送入判别器，提取中间层特征 (l-th layer)
feat_real = D.extract_feature(x)
feat_fake = D.extract_feature(x_recon)

# 相似性损失
recon_loss = F.mse_loss(feat_fake, feat_real)
```


#### 15. **MSSSIM-VAE**

* **核心**: 用 MS-SSIM (结构相似性) 替代 MSE Loss。适合人眼视觉。
* **关键代码**:

```python
# 计算 MS-SSIM
msssim = MS_SSIM(data_range=1.0)
sim_score = msssim(recon_x, x)

# Loss = 1 - SSIM (SSIM 越大越好)
return (1 - sim_score) + KLD
```

***

### 第六类：采样增强 (Importance Weighted)

*通过多次采样来获得更准的 ELBO。*

#### 16. **IWAE (Importance Weighted AE)**

* **核心**: 既然单次采样 $z$ 估计不准，那就采 $k$ 次，加权平均。
* **关键代码**:

```python
# 采样 k 次 z
z = reparameterize(mu, log_var, k=samples) # (B, K, D)

# 计算 Log 权重 w_i = p(x|z)p(z)/q(z|x)
log_w = log_p_x_z + log_p_z - log_q_z_x

# Log-Sum-Exp 技巧计算 Loss
loss = -torch.logsumexp(log_w, dim=1) + log(k)
```


#### 17. **MIWAE (Multiply IWAE)**

* **核心**: IWAE 的改进版，旨在得到更紧致的界。
* **关键代码**:
    * 逻辑类似 IWAE，但在梯度估算时使用了梯度的混合策略，使得在大 $K$ 值时梯度方差更小。

***

### 第七类：其他 (Others)

#### 18. **INFOVAE**

* **核心**: 显式最大化 $x$ 和 $z$ 的互信息 (MMD)，防止“Latent Ignore”问题（即解码器无视 $z$）。
* **关键代码**:

```python
# 使用 MMD 匹配 q(z) 和 p(z)
mmd = compute_mmd(z, torch.randn_like(z))
# 这里的 alpha * mmd 实际上在最大化互信息
return recon_loss + (1 - alpha) * KLD + (alpha + lambda) * mmd
```


#### 19. **WAE (Wasserstein AE)**

* **核心**: 使用最优传输距离（Wasserstein Distance）来匹配分布。实现上常用 MMD-GAN 或 WGAN 的思路。
* **关键代码**:

```python
# MMD 版本实现
# 匹配 聚合后验 (Aggregated Posterior) 和 先验
z_fake = encoder(x)
z_real = torch.randn_like(z_fake)

# 使用核函数 (RBF Kernel) 计算距离
mmd_dist = kernel_loss(z_fake, z_real)

return recon_loss + lambda * mmd_dist
```


