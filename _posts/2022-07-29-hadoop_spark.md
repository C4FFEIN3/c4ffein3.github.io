---
layout: post
title: "Spark에서 HDFS 파일 접근하기"
subtitle: "Spark를 이용해서 HDFS에 파일을 저장하고 읽어오기"
categories: hadoop-ecosystem
tags: [linux, ubuntu, hadoop-ecosystem, hdfs, spark]
---

## 하둡 실행

```bash
{hadoop의 root directory}/sbin/start-all.sh
```

## Spark를 통해 데이터를 HDFS에 쓰기

### writeSparkHDFS.py

```python
from pyspark.sql import SparkSession

sparkSession = SparkSession.builder.appName("example-pyspark-write").getOrCreate()

data = [('First', 1), ('Second', 2), ('Third', 3), ('Fourth', 4), ('Fifth', 5)]
df = sparkSession.createDataFrame(data)

df.write.csv("hdfs://localhost:9000/example.csv")
```

- **HDFS의 path**는 하둡의 core-site.xml 설정 파일에서 **fs.defaultFS**에 지정한 값
이 경우에는 hdfs://localhost:9000

### spark-submit을 통해 writeSparkHDFS.py 실행

```bash
spark-submit writeSparkHDFS.py
```

### HDFS shell을 통해 생성된 파일 확인

```bash
hadoop fs -ls /
```

![Untitled-00](https://user-images.githubusercontent.com/57282971/182781907-cc60efb8-382c-4234-9311-8071529842ce.png)

## Spark를 통해 데이터를 HDFS에서 읽기

### readSparkHDFS.py

```python
from pyspark.sql import SparkSession

sparkSession = SparkSession.builder.appName("example-pyspark-read").getOrCreate()

df = sparkSession.read.csv("hdfs://localhost:9000/user/example.csv")
df.show()
```

### spark-submit을 통해 readSparkHDFS.py 실행

```bash
spark-submit readSparkHDFS.py
```

![Untitled-01](https://user-images.githubusercontent.com/57282971/182781912-d10b6f79-86b6-4cdc-8a33-a9814c175ddc.png)
