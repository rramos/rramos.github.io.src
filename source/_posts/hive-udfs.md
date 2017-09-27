---
title: Hive UDFS functions
tags:
  - Hive
  - udfs
  - BigData
  - Hadoop
  - Java
  - Cloudera
date: 2017-09-27 21:16:37
---


Today i was going to use a simple sha256 funtion in [Hive](https://hive.apache.org/) in order to mask a colunm and aparently in the latest [Cloudera](https://www.cloudera.com/) distribution the Shipped hive version doesn't have that native function.

This article will explain how you can build a sha256 or other udfs function and add it in Hive.

# Checking Cloudera Packages Version

Check the following [URL](https://www.cloudera.com/documentation/enterprise/release-notes/topics/cdh_vd_cdh_package_tarball.html) in order to see the latest shipped package versions in Cloudera.

> CDH 5.12 -> hive-1.1.0+cdh5.12.1+1197

| Return Type| Name(Signature)           | Description          |
|------------|---------------------------|----------------------|
| string    | sha2(string/binary, int)   | Calculates the SHA-2 family of hash functions (SHA-224, SHA-256, SHA-384, and SHA-512) (as of Hive 1.3.0). The first argument is the string or binary to be hashed. The second argument indicates the desired bit length of the result, which must have a value of 224, 256, 384, 512, or 0 (which is equivalent to 256). SHA-224 is supported starting from Java 8. If either argument is NULL or the hash length is not one of the permitted values, the return value is NULL. Example: sha2('ABC', 256) = 'b5d4045c3f466fa91fe2cc6abe79232a1a57cdf104f7a26e716e0a1e2789df78'.|

[Full Version](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+UDF)

# Implement UDFS

this will server more as an exercise, one could create a more complex udf funtion. For the time being let's create a GenericUDFSha2 based on existing hive 1.3.0 version

You code clone my repo with some udfs-utils [here](git@github.com:rramos/udfs-utils.git)

The original code for hive version 1.3.0 is available in the repo

https://github.com/apache/hive/blob/master/ql/src/java/org/apache/hadoop/hive/ql/udf/generic/GenericUDFSha2.java

Let's create the building structure

```
mkdir -p GenericUDFSha2/org/apache/hadoop/hive/ql/udf/generic
cd GenericUDFSha2
```

Let's create a `pom.xml` file

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.rramos.bigdata.utils</groupId>
  <artifactId>GenericUDFSha2</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>GenericUDFSha2</name>
  <url>http://maven.apache.org</url>

  <!-- properties -->
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>
     
  <!-- prerequisitesprerequisites -->
  <prerequisites>
     <maven>3.0</maven>
  </prerequisites>

   <!-- Dependencies -->
   <dependencies>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.12</version>
      <scope>test</scope>
    </dependency>

    <dependency>
       <groupId>org.apache.hive</groupId>
       <artifactId>hive-exec</artifactId>
       <version>2.0.0</version>
    </dependency>

    <dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.9.3</version>
    </dependency>

  </dependencies>

  <!-- Build options -->
  <build>
   <plugins>
    <plugin>
     <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-jar-plugin</artifactId>
      <version>2.6</version>
       <configuration>
        <archive>
          <manifest>
           <addClasspath>true</addClasspath>
           <mainClass>com.rramos.bigdata.utils.GenericUDFSha2</mainClass>
          </manifest>
        </archive>
       </configuration>
      </plugin>

      <plugin>
          <artifactId>maven-assembly-plugin</artifactId>
          <configuration>
                <archive>
                    <manifest>
                        <mainClass>
                            com.rramos.bigdata.utils.GenericUDFSha2
                        </mainClass>
                    </manifest>
                </archive>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
           </configuration>
      </plugin>

    </plugins>
  </build>

</project>
```

You should obviously change for you packaging namespace, i'm just using `com.rramos.bigdata.utils` to be simpler.

Next, let's create the following file

`org/apache/hadoop/hive/ql/udf/generic/GenericUDFSha2.java`

with the content

```
package com.rramos.bigdata.utils;

import static org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorUtils.PrimitiveGrouping.BINARY_GROUP;
import static org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorUtils.PrimitiveGrouping.NUMERIC_GROUP;
import static org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorUtils.PrimitiveGrouping.STRING_GROUP;

import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

import org.apache.commons.codec.binary.Hex;
import org.apache.hadoop.hive.ql.exec.Description;
import org.apache.hadoop.hive.ql.exec.UDFArgumentException;
import org.apache.hadoop.hive.ql.exec.UDFArgumentTypeException;
import org.apache.hadoop.hive.ql.metadata.HiveException;
import org.apache.hadoop.hive.serde2.objectinspector.ConstantObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspector;
import org.apache.hadoop.hive.serde2.objectinspector.ObjectInspectorConverters.Converter;
import org.apache.hadoop.hive.serde2.objectinspector.PrimitiveObjectInspector.PrimitiveCategory;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorFactory;
import org.apache.hadoop.hive.serde2.objectinspector.primitive.PrimitiveObjectInspectorUtils;
import org.apache.hadoop.io.BytesWritable;
import org.apache.hadoop.io.Text;

/**
 * GenericUDFSha2.
 *
 */
@Description(name = "sha2", value = "_FUNC_(string/binary, len) - Calculates the SHA-2 family of hash functions "
    + "(SHA-224, SHA-256, SHA-384, and SHA-512).",
    extended = "The first argument is the string or binary to be hashed. "
    + "The second argument indicates the desired bit length of the result, "
    + "which must have a value of 224, 256, 384, 512, or 0 (which is equivalent to 256). "
    + "SHA-224 is supported starting from Java 8. "
    + "If either argument is NULL or the hash length is not one of the permitted values, the return value is NULL.\n"
    + "Example: > SELECT _FUNC_('ABC', 256);\n 'b5d4045c3f466fa91fe2cc6abe79232a1a57cdf104f7a26e716e0a1e2789df78'")
public class GenericUDFSha2 extends GenericUDF {
  private transient Converter[] converters = new Converter[2];
  private transient PrimitiveCategory[] inputTypes = new PrimitiveCategory[2];
  private final Text output = new Text();
  private transient boolean isStr;
  private transient MessageDigest digest;

  @Override
  public ObjectInspector initialize(ObjectInspector[] arguments) throws UDFArgumentException {
    checkArgsSize(arguments, 2, 2);

    checkArgPrimitive(arguments, 0);
    checkArgPrimitive(arguments, 1);

    // the function should support both string and binary input types
    checkArgGroups(arguments, 0, inputTypes, STRING_GROUP, BINARY_GROUP);
    checkArgGroups(arguments, 1, inputTypes, NUMERIC_GROUP);

    if (PrimitiveObjectInspectorUtils.getPrimitiveGrouping(inputTypes[0]) == STRING_GROUP) {
      obtainStringConverter(arguments, 0, inputTypes, converters);
      isStr = true;
    } else {
      GenericUDFParamUtils.obtainBinaryConverter(arguments, 0, inputTypes, converters);
      isStr = false;
    }

    if (arguments[1] instanceof ConstantObjectInspector) {
      Integer lenObj = getConstantIntValue(arguments, 1);
      if (lenObj != null) {
        int len = lenObj.intValue();
        if (len == 0) {
          len = 256;
        }
        try {
          digest = MessageDigest.getInstance("SHA-" + len);
        } catch (NoSuchAlgorithmException e) {
          // ignore
        }
      }
    } else {
      throw new UDFArgumentTypeException(1, getFuncName() + " only takes constant as "
          + getArgOrder(1) + " argument");
    }

    ObjectInspector outputOI = PrimitiveObjectInspectorFactory.writableStringObjectInspector;
    return outputOI;
  }

  @Override
  public Object evaluate(DeferredObject[] arguments) throws HiveException {
    if (digest == null) {
      return null;
    }

    digest.reset();
    if (isStr) {
      Text n = GenericUDFParamUtils.getTextValue(arguments, 0, converters);
      if (n == null) {
        return null;
      }
      digest.update(n.getBytes(), 0, n.getLength());
    } else {
      BytesWritable bWr = GenericUDFParamUtils.getBinaryValue(arguments, 0, converters);
      if (bWr == null) {
        return null;
      }
      digest.update(bWr.getBytes(), 0, bWr.getLength());
    }
    byte[] resBin = digest.digest();
    String resStr = Hex.encodeHexString(resBin);

    output.set(resStr);
    return output;
  }

  @Override
  public String getDisplayString(String[] children) {
    return getStandardDisplayString(getFuncName(), children);
  }

  @Override
  protected String getFuncName() {
    return "sha2";
  }
}
```


And build the package.

```
mvn package
```

After compile you find in `target` dir the require jar (`GenericUDFSha2-1.0-SNAPSHOT.jar`) you need to add in Hive.

You should use your Hadoop Distribution instructions for deploying new jars.

Here are [Cloudera Instructions](https://www.cloudera.com/documentation/enterprise/5-6-x/topics/cm_mc_hive_udf.html).

Next on your Hive session you need to `ADD JAR` and create a `FUNCTION` or `TEMPORARY FUNCTION`

```
    ADD JAR ./target/GenericUDFSha2-1.0-SNAPSHOT.jar
    CREATE TEMPORARY FUNCTION sha2 AS 'com.rramos.bigdata.utils.GenericUDFSha2';
    SELECT sha2(foo) from bar LIMIT1; 
```

[Matthew Rathbone](https://blog.matthewrathbone.com/2013/08/10/guide-to-writing-hive-udfs.html) Blog has some great tutorial on Hive Funtions. Take a look if you want to go deep with it.

Cheers,
RR

# References

* https://www.cloudera.com/documentation/enterprise/release-notes/topics/rg_cdh_vd.html
* https://www.cloudera.com/documentation/enterprise/5-6-x/topics/cm_mc_hive_udf.html
* https://blog.matthewrathbone.com/2013/08/10/guide-to-writing-hive-udfs.html