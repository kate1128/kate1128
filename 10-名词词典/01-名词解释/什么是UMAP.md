<!-- type: glossary -->
# UMAP（Uniform Manifold Approximation and Projection）

> 一种用于**高维数据降维和可视化**的算法。相比经典的 t-SNE，UMAP 在保留数据全局结构方面更优，计算速度也更快。

## 是什么

在机器学习和数据科学中，很多数据是高维的（比如 Embedding 向量通常是 1536 或 3072 维）。人无法直观"看到"高维空间。UMAP 的作用就是把这些高维数据**投影到 2D 或 3D 平面**上，同时尽可能保留原始数据中的聚类关系。

适用场景：
- **向量可视化**：把 Embedding 向量画在 2D 平面上，看语义聚类是否合理（比如"猫""狗""汽车"是否各自成簇）
- **数据探索**：在训练模型前，先用 UMAP 看看数据分布——有没有离群点？类别是否可分离？
- **调试 RAG**：查看检索结果是否真的在向量空间中靠近查询

## 为什么重要

- **Embedding 质量检查的利器**：向量模型好不好，先用 UMAP 画个图——同类是否聚在一起，不同类是否分得开
- **t-SNE 的替代品**：t-SNE 保留了局部结构但扭曲了全局结构；UMAP 在保留全局关系上表现更好
- **速度快**：UMAP 比 t-SNE 快一个数量级，处理十万级别的数据点完全可行

## 关键要点

- **安装**：`pip install umap-learn`（注意包名是 `umap-learn` 不是 `umap`）
- **关键参数**：`n_neighbors`（控制局部/全局平衡，默认 15）、`min_dist`（控制点的紧密程度，默认 0.1）、`n_components`（输出维度，可视化为 2 或 3）
- **与 t-SNE 的选择**：t-SNE 适合可视化局部聚类；UMAP 适合需要保留全局结构的场景

## 相关概念

- [[Embedding 词向量]] — UMAP 最常见的应用场景：可视化 Embedding 向量
- [[PCA 主成分分析]] — 另一种经典降维方法，线性投影，速度更快但表达能力不如 UMAP
- [[t-SNE]] — UMAP 的前身和主要对比对象

## 参考

- [UMAP 论文](https://arxiv.org/abs/1802.03426)
- [UMAP 官方文档](https://umap-learn.readthedocs.io/)