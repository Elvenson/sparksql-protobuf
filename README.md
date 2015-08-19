# sparksql-protobuf

This library provides utilities to work with Protobuf objects in SparkSQL.
It provides a way to read parquet file written by SparkSQL back as an RDD of compatible protobuf object.
It can also converts RDD of protobuf objects into DataFrame.

[![Build Status](https://travis-ci.org/saurfang/sparksql-protobuf.svg?branch=master)](https://travis-ci.org/saurfang/sparksql-protobuf)
[![codecov.io](https://codecov.io/github/saurfang/sparksql-protobuf/coverage.svg?branch=master)](https://codecov.io/github/saurfang/sparksql-protobuf?branch=master)

For sbt 0.13.6+

```scala
resolvers += Resolver.jcenterRepo

libraryDependencies ++= Seq(
    "com.github.saurfang" %% "sparksql-protobuf" % "0.1.2",
    "org.apache.parquet" % "parquet-protobuf" % "1.7.0"
)
```

Note you might want to change `parquet-protobuf` depends on what version of Spark you're using. Spark 1.4 and prior uses
`parquet` under `com.twitter` thus causes no conflict and you can pick whichever version you prefer. Spark 1.5 is scheduled
to use `"org.apache.parquet" % "parquet-hadoop" % "1.7.0"` so you will probably want to be consistent.

## Motivation

SparkSQL is very powerful and easy to use. However it has a few limitations and schema is only detected during runtime
makes developers a lot less confident that they will get things right at first time. This is where protobuf comes in:

1. Protobuf defines nested data structure easily 
2. It doesn't constraint you to the 22 fields limit in case class (no longer true once we upgrade to 2.11+)
3. It is language agnostic and generates code that gives you native object 
hence you get all the benefit of type checking and code completion unlike operating `Row` in Spark/SparkSQL

## Feature
1. `ProtoMessageConverter` has been improved to read from [LIST specification](https://github.com/apache/parquet-format/blob/master/LogicalTypes.md#lists)
according to latest parquet documentation. This implementation should be backwards compatible and is able to read repeated
fields generated by writers like SparkSQL.
2. `ProtoMessageParquetInputFormat` helps the above process by correctly returning the built protobuf object as value.
3. `ProtoParquetRDD` abstract the Hadoop input format and returns an RDD of your protobuf objects from parquet files directly.
4. `ProtoReflection` infers SparkSQL schema from any Protobuf message class.
5. `ProtoRDDConversions` converts Protobuf objects into SparkSQL rows.

## Usage

### Read Parquet file as `RDD[Protobuf]`

```scala
val personsPB = new ProtoParquetRDD(sc, "persons.parquet", classOf[Person])
```

where we need `SparkContext`, parquet path and protobuf class.

### Infer SparkSQL Schema from Protobuf Definition

```scala
val personSchema = ProtoReflection.schemaFor[Person].dataType.asInstanceOf[StructType]
```

### Convert `RDD[Protobuf]` to `DataFrame`

```scala
import com.github.saurfang.parquet.proto.spark.sql._
val personsDF = sqlContext.createDataFrame(protoPersons)
```

*For more information, please see test cases.*

## Related Work
[Elephant Bird](https://github.com/twitter/elephant-bird)
