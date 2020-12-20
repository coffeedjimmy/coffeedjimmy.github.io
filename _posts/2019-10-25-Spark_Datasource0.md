---
layout: post
title: "Spark 데이터 소스"
tags: [spark]
comments: true
---

.s

- 스파크에는 6개의 핵심 데이터 소스와 커뮤니티에서 만든 다양한 외부 데이터 소스들이 존재.
    - 핵심 데이터 소스
        - CSV
        - JSON
        - Parquet
        - ORC
        - JDBC/ODBC 연결
        - 일반 텍스트 파일
    - 외부 데이터 소스
        - 카산드라
        - HBase
        - MongoDB

## 9.1. 데이터 소스 API의 구조

읽기 API 구조

    DataFrameReader.format(...).option("key", "value").schema(...).load()

- format() 은 선택적으로 사용할 수 있으며, 기본은 Parquet
- 스파크에서 데이터를 읽을 때 기본적으로 `DataFrameReader` 를 사용.
    - DataFrameReader는 [spark.read](http://spark.read) 속성으로 접근

    spark.read.format("csv")
    	.option("mode", "FAILFAST")
    	.option("inferSchema", "true")
    	.option("path", "path/to/file(s)")
    	.schema(someSchema)
    	.load()

**읽기모드**

- permissive: 오류 레코드의 모든 필드를 null로 설정하고 모든 오류 레코드를 _corrupt_record 컬럼에 기록 (기본값)
- dropMalformed: 오류 레코드를 제거
- failFast : 오류 레코드가 있으면 즉시 종료

쓰기 API 구조

    DataFrameWriter.format(...)
    	.option(...).partitionBy(...).bucketBy(...).sortBy(...).save()

- format()은 선택적으로 사용할 수 있으며 기본값은 Parquet
- partitionedBy, bucketBy, sortBy는 파일 기반의 데이터 소스에서만 동작
    - 이 기능을 활용해 최종 파일 배치 형태를 제어할 수 있음.
- DataFrameWriter는 dataFrame.write 속성으로 접근

    dataframe.write.format("csv")
    	.option("mode", "OVERWRITE")
    	.option("dataFormat", "yyyy-MM-dd")
    	.option("path", "path/to/file(s)")
    	.save()

**저장모드**

- append: 해당 경로에 이미 존재하는 파일 목록에 결과 파일을 추가
- overwrite: 데이터 덮어쓰기
- errorIfExists: 이미 존재하고 있으면 에러 (기본값)
- ignore: 그냥 무시

## 9.2 CSV 파일

파일 읽기

    spar.read.format("csv")
    	.option("header", "true")
    	.option("mode", "FAILFAST")  // 비정상적인 데이터를 어느정도 수용할지.
    	.option("inferSchema", "true")
    	.load("some/path/to/file.csv")

- Spark는 LazyExecution 을 활용하므로 실제 잡 실행시점에만 오류 발생.

파일 쓰기

    csvFile.write.format("csv").mode("overwrite").option("sep", "\t").save("abc.tsv")

## 9.3 JSON 파일

파일 읽기

    spark.read.format("json").option("mode", "FAILFAST").schema(mySchema)
    	.load("abc.json").show(5)

파일 쓰기

    csvFile.write.format("json").mode("overwrite").save("abc.json")

## 9.4. Parquet 파일

- Parquet는 다양한 스토리지 최적화 기술을 제공하는 오픈소스로 제작된 컬럼 기반의 데이터 저장 방식.
- 전체 파일을 읽는 대신 개별 컬럼을 읽을 수 있음.
- 컬럼 기반의 압축 제공
- 스파크와 호환이 잘 되어 스파크의 기본 파일 포맷
- 읽기 연산에 있어 JSON이나 CSV 보다 훨씬 효율적으로 동작 → 장기 저장용 데이터는 Parquet로 저장하는 것을 권장

파일 읽기

    spark.read.format("parquet")
    	.load("abc.parquet").show(5)

파일 쓰기

    csvFile.write.format("parquet").mode("overwrite")
    	.save("abc.parquet")

## 9.5 ORC 파일

- 하둡 워크로드를 위해 설계된 데이터 타입을 인식할 수 있는 컬럼 기반의 파일 포맷.
- 대규모 스트리밍 읽기에 최적화.
- 필요한 row를 신속하게 찾아낼 수 있는 기능이 통합.
- Parquet와 유사하지만 Parquet는 스파크에 최적화되어 있고, ORC는 하둡 최적화.

파일 읽기

    spark.read.format("orc").load("aaa.orc").show(5)

파일 쓰기

    csvFile.write.format("orc").mode("overwrite").save("aaa.orc")

## 9.6 SQL 데이터 베이스

- 데이터베이스는 원시 파일 형태가 아니기 때문에 고려해야할 점이 더 많음.
    - 인증정보, 접속정보 등
- 데이터베이스의 데이터를 읽고 쓰기 위해서는 spark classpath에 데이터베이스의 JDBC 드라이버를 추가하고 그에 맞는 JDBC 드라이버 jar 파일을 제공해야 합니다.

    ./bin/spark-shell \
    --driver-class-path postgresql-9.4.1207.jar \
    --jars postgresql-9.4.1207.jar

데이터베이스 읽기

    val driver = "org.sqlite.JDBC"
    val path = "/data/flight-data/jdbc/my-sqlite.db"
    val url = s"jdbc:sqlite:/${path}"
    val tablename = "flight_info"
    
    // 접속 테스트
    import java.sql.DriverManager
    
    val connection = DriverManager.getConnection(url)
    connection.isClosed()
    connection.close()
    
    // 데이터베이스 읽기 (sqlite)
    val dbDataFrame = spark.read.format("jdbc").option("url", url)
    	.option("dbtable", tablename).option("driver", driver).load()

쿼리 푸시다운

- 스파크는 DataFrame을 만들기 전에 데이터베이스 자체에서 데이터를 필터링하도록 만들 수 있음.

    dbDataFrmae.filter("DEST_COUNTRY_NAME in ('Anguilla', 'Sweden')").explain

- 필터를 명시하면 스파크는 해당 필터에 대한 처리를 데이터베이스로 위임(push down) 함.
- 스파크가 모든 스파크 함수를 사용하는 SQL 데이터베이스에 맞게 변환하지는 못함.

    val pushdownQuery = """(SELECT DISTINCT(DEST_COUNTRY_NAME) FROM flight_info)
    AS flight_info"""
    
    val dbDataFrame = spark.read.format("jdbc")
    	.option("url", url).option("dbtable", pushdownQuery).option("driver", driver)
    	.load()

데이터베이스 병렬로 읽기

- 스파크는 파일 크기, 파일 유형 그리고 압축 방식에 따른 '분할 가능성'에 따라 여러 파일을 읽어 하나의 파티션으로 만들거나 여러 파티션을 하나의 파일로 만드는 기본 알고리즘을 가지고 있음.

    val dbDataFrame = spark.read.format("jdbc")
    	.option("url", url).option("dbtable" tablename).option("driver", driver)
    	.option("numPartitions", 10).load()  // 동시 작업 수 제한.

데이터베이스 쓰기

    val newPath = "jdbc:sqlite://tmp/my-sqlite.db"
    csvFile.write.mode("overwrite").jdbc(newPath, tablename, props)

## 9.7. 텍스트 파일

- 파일의 각 줄이 DataFrame의 레코드가 됨.

파일 읽기

    spark.read.textFile("/data/flight/csv/2019.csv")
    	.selectExpr("split(value, '_') as rows").show()

파일 쓰기

    // 문자열 컬럼이 하나만 존재해야 함.
    csvFile.select("DEST_COUNTRY_NAME").write.text("/tmp/simple.txt")
    
    // 파티셔닝 작업을 수행하면 더 많은 컬럼을 저장할 수 있지만,
    // 모든 파일에 컬럼을 추가하는 것이 아니라 텍스트 파일이 저장되는 디렉터리에 폴더별로 컬럼 저장
    
    csvFile.limit(10).select("DEST_COUNTRY_NAME", "count")
    	.write.partitionBy("count").text("/tmp/test.csv")