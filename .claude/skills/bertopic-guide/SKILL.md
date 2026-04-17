---
name: bertopic-guide
description: BERTopic dynamic topic modeling on large-scale Chinese text data with memory-constrained machines (8GB RAM). Use when setting up BERTopic pipelines, optimizing memory for large datasets, or troubleshooting performance issues with UMAP/HDBSCAN/embeddings.
user-invocable: true
argument-hint: "[task description or question]"
---

# BERTopic 大规模中文文本动态主题建模指南

基于 200 万条中文社交媒体数据、Apple M1 8GB RAM 的实战经验总结。

## 一、核心流程

```
原始文本 → 清洗 → 嵌入(768维) → PCA(50维) → UMAP(5维) → HDBSCAN → c-TF-IDF → 主题表示 → 动态主题
```

## 二、8GB 内存优化策略（关键！）

### 1. 嵌入计算：分 chunk 保存 + 断点续算
- **问题**：预分配 5.8GB memmap 导致 swap 暴涨至 14GB，CPU 降到 0.2%
- **方案**：每 10K 条文档为一个 chunk，独立保存 .npy 文件（~30MB/chunk）
- 完成后用 memmap 拼接为合并文件，以只读 mmap 加载
- 支持断点续算：已完成的 chunk 自动跳过

```python
CHUNK_SIZE = 10000
for chunk_idx in range(n_chunks):
    if chunk_idx in done_chunks:
        continue
    chunk_emb = model.encode(chunk_docs, batch_size=64, device="mps")
    np.save(chunk_path, chunk_emb)
    del chunk_emb; gc.collect()
```

### 2. PCA 降维：768 → 50
- **问题**：2M × 768 × float32 = 6.2GB，无法放入 8GB 内存
- **方案**：IncrementalPCA 逐批处理，输出 2M × 50 = 405MB
- 解释方差 ~55% 是 BERT 类嵌入的正常水平（高维中大量冗余信息）
- 对主题建模质量影响极小：Top 50 主成分足以捕获主题级语义差异

```python
from sklearn.decomposition import IncrementalPCA
ipca = IncrementalPCA(n_components=50, batch_size=10000)
for i in range(0, n_docs, 10000):
    ipca.partial_fit(embeddings[i:i+10000])
```

### 3. UMAP 子采样：fit 20 万 + batch transform 全量
- **问题**：UMAP 在 200 万点上构建图结构导致 OOM 被 kill
- **方案**：随机采样 20 万条 fit，再分批 transform 全量（5 万/批）
- UMAP 作者推荐 >50 万数据时采用子采样策略
- 质量影响：200K 样本足以捕获全局拓扑结构

```python
sample_idx = np.random.RandomState(42).choice(n_docs, 200000, replace=False)
umap_model.fit(pca_embeddings[sample_idx])
for batch in batches(50000):
    umap_embeddings[s:e] = umap_model.transform(pca_embeddings[s:e])
```

### 4. PrecomputedUMAP + PrecomputedHDBSCAN 透传类
- **问题**：BERTopic.fit_transform() 内部会重新运行 UMAP 和 HDBSCAN
- **方案**：创建透传类，直接返回预计算的结果

```python
class PrecomputedUMAP:
    def __init__(self, embeddings):
        self._emb = embeddings
    def fit(self, X, y=None): return self
    def transform(self, X): return self._emb
    def fit_transform(self, X, y=None): return self._emb

class PrecomputedHDBSCAN:
    def __init__(self, labels):
        self.labels_ = labels
    def fit(self, X, y=None): return self
    def predict(self, X): return self.labels_
    def fit_predict(self, X, y=None): return self.labels_
```

### 5. 激进内存管理
- **尽早释放 DataFrame**：提取 docs/timestamps 后只保留轻量列
- **跳过不必要的模型加载**：嵌入缓存存在时不加载 SentenceTransformer（省 ~400MB）
- **每步缓存**：embeddings, PCA, UMAP, HDBSCAN labels 全部缓存为 .npy 文件
- **及时 del + gc.collect()**：每个大对象用完立即释放
- **HDBSCAN 不要设 prediction_data=True**：浪费内存，本场景不需要

### 6. 缓存策略一览

| 缓存文件 | 内容 | 大小 |
|---------|------|------|
| `emb_chunks/chunk_XXXX.npy` | 分块嵌入 | ~30MB/chunk |
| `embeddings_combined.dat` | 合并嵌入 memmap | 6.2GB |
| `embeddings_pca50.npy` | PCA 降维结果 | 405MB |
| `umap_embeddings_5d.npy` | UMAP 降维结果 | 40MB |
| `hdbscan_labels.npy` | 聚类标签 | 8MB |

## 三、中文主题表示优化

### 问题
默认 CountVectorizer + jieba 会把英文碎片、数字、符号也当作有效词元，导致主题表示全是噪声。

### 方案：分词过滤 + KeyBERT + MMR

```python
# 1. 分词器只保留 ≥2 个中文字符的词
def jieba_tokenizer(text):
    return [w for w in jieba.cut(text)
            if len(w) >= 2 and re.search(r'[\u4e00-\u9fff]{2}', w)]

# 2. 链式表示模型
from bertopic.representation import KeyBERTInspired, MaximalMarginalRelevance
representation_model = [
    KeyBERTInspired(top_n_words=30),         # 语义精排
    MaximalMarginalRelevance(diversity=0.3),  # 多样性去重
]

# 3. 传给 BERTopic
topic_model = BERTopic(
    embedding_model=SentenceTransformer("shibing624/text2vec-base-chinese"),
    representation_model=representation_model,
    ...
)
```

## 四、关键教训

### Swap 陷阱
- 8GB Mac 上，2M 中文字符串 + DataFrame 可轻松占 4-5GB
- HDBSCAN Boruvka 算法对 swap 极度敏感：内存不足时 CPU 从 99% 降到 0.2%
- **解法**：在运算密集步骤前尽量释放内存到 3-4GB swap 以下

### BERTopic 内部行为
- `fit_transform()` 会**重新运行** UMAP 和 HDBSCAN，即使你已经 fit 过
- 必须用 PrecomputedUMAP / PrecomputedHDBSCAN 透传类阻止重复计算
- `topics_over_time()` 会对每个时间 bin 重新做分词 + c-TF-IDF，非常耗时

### 多实例问题
- 用户可能不小心启动多个实例，导致多个模型争夺 GPU 和内存
- 每次运行前先检查 `ps aux | grep` 确保没有重复进程
- 使用 `nohup ... &` 后台运行，断网不影响

### 可视化注意
- 层级图（visualize_hierarchy）在主题数 >100 时容易出现 "Distance matrix cannot contain negative values" 错误
- **解法**：限制为 Top 50 主题 `topic_model.visualize_hierarchy(topics=top_50_topics)`

## 五、推荐硬件与参数

### 8GB RAM 推荐参数

| 参数 | 推荐值 | 说明 |
|------|--------|------|
| CHUNK_SIZE | 10,000 | 每 chunk ~30MB |
| BATCH_SIZE | 64 | MPS GPU 编码 |
| PCA_DIM | 50 | 768→50 降维 |
| UMAP n_components | 5 | 降到 5 维 |
| UMAP_SAMPLE | 200,000 | fit 采样量 |
| UMAP_TRANSFORM_BATCH | 50,000 | transform 批大小 |
| HDBSCAN min_cluster_size | 50 | 最小簇大小 |
| HDBSCAN min_samples | 15 | 核心点密度 |
| CountVectorizer min_df | 10 | 最低词频 |

### 16GB+ RAM 可以放宽
- UMAP 可直接 fit 全量（无需子采样）
- PCA 可跳过或用更高维度（如 100）
- HDBSCAN 可设 prediction_data=True

## 六、典型运行时间（M1 8GB, 2M 文档）

| 步骤 | 首次运行 | 缓存加载 |
|------|---------|---------|
| 数据加载 + 预处理 | 3 min | 3 min |
| 嵌入（MPS GPU） | 8-9 hours | 秒级 |
| PCA | 5 min | 秒级 |
| UMAP | 12 min | 秒级 |
| HDBSCAN | 7 min | 秒级 |
| BERTopic + KeyBERT + MMR | 55 min | 需重跑 |
| 动态主题（12 月份） | 135 min | 需重跑 |
| 可视化 | 1 min | 1 min |

## 七、使用时的注意事项

当用户请求帮助进行 BERTopic 建模时：

1. **先问数据规模和硬件**：文档数量、内存大小决定优化策略
2. **先跑嵌入**：这是最耗时的步骤，优先确保有断点续算
3. **逐步验证**：每步完成后检查 CPU/内存/swap，发现瓶颈立即优化
4. **缓存一切**：每个计算密集步骤的结果都应该缓存到磁盘
5. **监控工具**：`ps -o pid,rss,%cpu`, `vm_stat`, `sysctl vm.swapusage`, `sample PID 1`
