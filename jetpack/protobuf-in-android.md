# Support protobuf in Andorid
## Overview
> Protocol buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, 
> but smaller, faster, and simpler. You define how you want your data to be structured once, then you can use special generated source 
> code to easily write and read your structured data to and from a variety of data streams and using a variety of languages.

Google protobuf，一种可以序列化的数据结构，相比于json、xml更小，同时由于是字节流数据，抓包时，也更不容易被阅读。用户定义一个```.proto```文件结构，protobuf plugin会帮助你自动
生成对应语言的类，用户通过类来操作数据。

## Guides

Android上使用例子如下，
在project的build.gradle文件下，增加如下内容
```
    repositories {
        //protobuf 插件在Maven Central上
        mavenCentral()
    }
    dependencies {
        //protobuf插件
        classpath "com.google.protobuf:protobuf-gradle-plugin:0.8.18"
    }
```
在app的build.gradle文件下，增加如下内容，
```
plugins {
    //导入插件
    id "com.google.protobuf"
}
```
```
android {
    //指定.proto文件存放位置
    sourceSets {
        main {
            proto {
                // In addition to the default 'src/main/proto'
                srcDir 'src/main/proto'
                srcDir 'src/main/protocolbuffers'
                // In addition to the default '**/*.proto' (use with caution).
                // Using an extension other than 'proto' is NOT recommended,
                // because when proto files are published along with class files, we can
                // only tell the type of a file from its extension.
                include '**/*.proto'
            }
//            java {
//                ...
//            }
        }
        test {
            proto {
                // In addition to the default 'src/test/proto'
                srcDir 'src/test/protocolbuffers'
            }
        }
    }
}
```
```
//https://github.com/google/protobuf-gradle-plugin
//生成java源文件，并添加到编译环境中
protobuf {
    protoc {
        artifact = 'com.google.protobuf:protoc:3.5.1'
    }
    plugins {
        javalite {
            artifact = 'com.google.protobuf:protoc-gen-javalite:3.0.0'
        }
    }
    generateProtoTasks {
        all()*.plugins {
            javalite { }
        }
    }
}
```

```
dependencies {
    //protobuf
    api 'com.google.protobuf:protobuf-lite:3.0.1'
    api 'com.google.protobuf:protoc:3.5.1'
    api 'javax.annotation:javax.annotation-api:1.3.2'
}
```
同时在main/proto目录下新建一个.proto文件，内容如下，build后，proto插件会在app的build/generated/source/proto目录下生成相应的class文件，我们的代码是通过生成的class文件来操作的。
```
syntax = "proto3";

option java_outer_classname = "HelloClass";

// The greeting service definition.
//service Greeter {
//  // Sends a greeting
//  rpc SayHello (HelloRequest) returns (HelloReply) {}
//}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
  uint32 age = 2;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
  int32 code = 2;
}
```
代码中操作方法如下，
```
fun main(args: Array<String>) {
    val helloClass = HelloClass
        .HelloReply
        .newBuilder()
        .setCode(100)
        .setMessage("ok").build()

    //serialize
    val bytes = helloClass.toByteArray()
    //deserialize
    val parseHelloClass = HelloClass.HelloReply.parseFrom(bytes)

    println("bytes ${bytes.size} $bytes")
    println("${helloClass.code} ${helloClass.message}")
    println("${parseHelloClass.code} ${parseHelloClass.message}")
}
```
