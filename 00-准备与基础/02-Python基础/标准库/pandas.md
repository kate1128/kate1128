`pandas` 是 Python 中一个非常流行的库，用于数据处理和分析。它提供了强大的数据结构和工具，比如 **DataFrame** 和 **Series**，可以高效地处理结构化数据。以下是如何使用 `pandas` 的一个完整指南。

---

### **安装 pandas**
如果尚未安装，可以通过以下命令安装：

```bash
pip install pandas
```

---

### **引入 pandas**
在 Python 脚本中，通常通过以下方式导入：

```python
import pandas as pd
```

---

### **pandas 的核心数据结构**
1. **Series**：一维数据结构，类似于带索引的数组。
2. **DataFrame**：二维数据结构，类似于电子表格或数据库表格。

---

### **如何使用 pandas**
#### 1. **创建数据**
你可以通过多种方式创建 `Series` 和 `DataFrame`。

##### **创建 Series**
```python
import pandas as pd

# 从列表创建 Series
s = pd.Series([10, 20, 30, 40])
print(s)

# 自定义索引
s = pd.Series([10, 20, 30, 40], index=["a", "b", "c", "d"])
print(s)
```

##### **创建 DataFrame**
```python
# 从字典创建 DataFrame
data = {
    "Name": ["Alice", "Bob", "Charlie"],
    "Age": [25, 30, 35],
    "City": ["New York", "Los Angeles", "Chicago"]
}

df = pd.DataFrame(data)
print(df)
```

---

#### 2. **读取数据**
`pandas` 可以读取多种格式的数据文件，比如 CSV、Excel、SQL、JSON 等。

##### **读取 CSV 文件**
```python
df = pd.read_csv("data.csv")
print(df)
```

##### **读取 Excel 文件**
```python
df = pd.read_excel("data.xlsx")
print(df)
```

##### **读取 JSON 文件**
```python
df = pd.read_json("data.json")
print(df)
```

---

#### 3. **基本操作**
##### **查看数据**
```python
# 查看前几行数据
print(df.head())

# 查看后几行数据
print(df.tail())

# 数据摘要信息
print(df.info())

# 描述性统计
print(df.describe())
```

##### **选择列**
```python
# 选择单列
print(df["Name"])

# 选择多列
print(df[["Name", "Age"]])
```

##### **选择行**
```python
# 按索引选择
print(df.iloc[0])  # 第一行
print(df.iloc[1:3])  # 第 2 到第 3 行

# 按标签选择
print(df.loc[0])  # 第一行
```

##### **筛选数据**
```python
# 筛选年龄大于30的行
filtered_df = df[df["Age"] > 30]
print(filtered_df)
```

##### **添加新列**
```python
df["Country"] = ["USA", "USA", "USA"]
print(df)
```

##### **修改列**
```python
df["Age"] = df["Age"] + 5  # 年龄增加 5
print(df)
```

##### **删除列**
```python
df = df.drop("Country", axis=1)
print(df)
```

---

#### 4. **数据清洗**
##### **处理缺失值**
```python
# 检查缺失值
print(df.isnull())

# 删除包含缺失值的行
df = df.dropna()

# 填充缺失值
df = df.fillna(0)
```

##### **重命名列**
```python
df.rename(columns={"Name": "Full Name", "Age": "Years"}, inplace=True)
print(df)
```

##### **修改索引**
```python
df.set_index("Name", inplace=True)
print(df)
```

---

#### 5. **数据分析**
##### **基本统计**
```python
print(df["Age"].mean())  # 平均值
print(df["Age"].sum())   # 总和
print(df["Age"].max())   # 最大值
print(df["Age"].min())   # 最小值
```

##### **分组操作**
```python
# 按列分组并计算统计信息
grouped = df.groupby("City")["Age"].mean()
print(grouped)
```

---

#### 6. **导出数据**
##### **导出为 CSV 文件**
```python
df.to_csv("output.csv", index=False)
```

##### **导出为 Excel 文件**
```python
df.to_excel("output.xlsx", index=False)
```

---

### **完整示例**
以下是一个从创建数据到分析数据的完整示例：

```python
import pandas as pd

# 创建数据
data = {
    "Name": ["Alice", "Bob", "Charlie", "David"],
    "Age": [25, 30, 35, 40],
    "City": ["New York", "Los Angeles", "Chicago", "Houston"]
}
df = pd.DataFrame(data)

# 查看数据
print(df.head())

# 筛选年龄大于 30 的人
filtered_df = df[df["Age"] > 30]
print(filtered_df)

# 添加新列
df["Country"] = "USA"
print(df)

# 统计城市中年龄的平均值
grouped = df.groupby("City")["Age"].mean()
print(grouped)

# 导出数据
df.to_csv("output.csv", index=False)
```

---

### **学习资源**
如果你想更深入学习 `pandas`，可以参考：

+ 官方文档：[pandas Documentation](https://pandas.pydata.org/docs/)
+ 在线课程：[Kaggle pandas Tutorial](https://www.kaggle.com/learn/pandas)
+ 书籍：《Python 数据分析实战》

`pandas` 是一个非常强大的工具，随着实践，你会发现更多高级功能，比如时间序列分析、数据透视表、多索引操作等。

