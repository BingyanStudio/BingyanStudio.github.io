---
title: 深入理解Attention
cover_image: /img/cover.png
abbrlink: f6673104
date: 2024-06-06 07:04:04
---

目的：输入一个sequence, 我们预测的下一个字符要与前面的sequence都有关联，不能只看前一个字符来预测下一个。是communication mechanism。

最直观的想法：把前面的token加起来做平均

做平均等价于矩阵乘法：

```
# x 为输入的序列[0.33, 0.33, 0.33] @ x# T 为time，即序列长度# 对w的行做平均w = torch.ones(T, T)w = w / w.sum(1, keepdim=True)w @ x
```
又因为序列上每个位置的字符只能看到前面的字符，不能看到后面的（这是CausalSelfAttention）

```
# w 是下三角矩阵w = torch.tril(torch.ones(T, T))w = w / w.sum(1, keepdim=True)w @ x
```
等价于：

```
tril = torch.tril(torch.ones(T, T))w = torch.zeros((T, T))w = w.masked_fill(tril==0, float('-inf'))w = F.softmax(w, dim = -1)w @ x
```
其中

如果需要整个序列上的字符都能communicate to each other（encoder block），去掉masked_fill(tril==0, float('-inf'))就行

那么w如何得到呢？

w要使得sequence中的token能够communicate，我们可以用矩阵乘法来实现

```
key = nn.Linear(C, head_size, bias=False)query = nn.Linear(C, head_size, bias=False)# x = (B, T, C)k = key(x) # (B, T, head_size)q = query(x) # (B, T, head_size)w = q @ k.transpose(-2, -1)  # (B, T, head_size) @ (B, head_size, T) --> (B, T, T)
```
又发现如果直接用q@k.transpose(-2, -1)得到w，w的值过大，我们需要它接近1，则用实现，dk是head_size

那既然使w接近1了，为什么还有用softmax呢？

softmax可以让较大的值更明显，让diffuse的值变converge

x也进行Linear操作，得到value

最终版为：

```
key = nn.Linear(C, head_size, bias=False)query = nn.Linear(C, head_size, bias=False)# x = (B, T, C)k = key(x) # (B, T, head_size)q = query(x) # (B, T, head_size)w = q @ k.transpose(-2, -1) * head_size ** -0.5  # (B, T, head_size) @ (B, head_size, T) --> (B, T, T)w = w.masked_fill(tril==0, float('-inf'))w = F.softmax(w, dim = -1)value = nn.Linear(C, head_size, bias=False)v = value(x)out = w @ v
```
注意：

* 是每个batch里进行attention, 不同的batch无法communicate
* self-attention指key、value与query从同一个x中产生，而cross-attention指query从x中产生，而key、value从其他地方产生。translation中，encode后的数据加入到decoder（cross-attention）,使得在翻译过程中，不仅能看到前面的信息，还能看到整个句子链接的信息。

