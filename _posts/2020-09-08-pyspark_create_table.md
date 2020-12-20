---
title: Pyspark로 하이브테이블 생성하기
date: 2020.09.08 12:00:00
categories:
- spark
tags:
- spark
- pyspark
---

Pyspark를 활용해 Hive테이블 구성하는 방법을 다룹니다.


Pyspark를 활용하여 대규모 데이터를 lazy execution으로 처리한 뒤, 해당 결과를 hive 테이블로 만들고 싶은 경우가 있을 수 있습니다.

이런 경우 다음과 같은 방식으로 하이브테이블을 구성하고, 데이터 insert를 진행할 수 있습니다.

```python
if not spark._jsparkSession.catalog().tableExists('TARGET_TABLE'):
    df.write.partitionBy('partition1', 'partition2').\
        saveAsTable('TARGET_TABLE')
else:
    df.write.mode('overwrite').\
        insertInto('TARGET_TABLE')
```

위의 방식에서 다소 생소한 `_jsparkSession`을 통한 카탈로그 접근을 볼 수 있는데, 이는 spark 2.2 대의 버전을 위한 방식입니다.

spark 2.3부터는 `spark.catalog.tableExists`와 같은 방식으로 테이블 존재여부를 체크할 수 있습니다.

.
