---
title: "Issue with Jackson serialization of isGetter with continuous capital letters"
excerpt_separator: "<!--more-->"
categories:
  - Issue and Workaround
tags:
  - Jackson
  - Kotlin
  - Json
---

Noticed a small issue while using jackson to serialize a data class with a property that is a "is-getter" and has continuous capital letters after the "is". For example, the following class 

```
data class SampleWithIsGetter (
    val isUSDListing: Boolean
)
```

will serialized as 

```
{ "usdlisting" : true }
```

instead of 

```
{ "isUSDListing" : true }
```

The workaround is to set the MapperFeature - [USE_STD_BEAN_NAMING](https://fasterxml.github.io/jackson-databind/javadoc/2.6/com/fasterxml/jackson/databind/MapperFeature.html#USE_STD_BEAN_NAMING) of the ObjectMapper to true. But as of jackson 2.13, the configuration of MapperFeature from within the ObjectMapper has been deprecated, so this feature has to be set in the builder of the objectmapper. A sample of the workaround is as below

```
    val sample = SampleWithIsGetter(true)
    val objectMapper = jacksonMapperBuilder().configure(MapperFeature.USE_STD_BEAN_NAMING, true).build()
    val json = objectMapper.writeValueAsString(sample)
    val deserialized = objectMapper.readValue(json, SampleWithIsGetter::class.java)
    println(json)
    assertThat(sample, equalTo(deserialized))
```

Full description with both failure case and workaround is available on [https://github.com/thecodinganalyst/jackson-kotlin-test](https://github.com/thecodinganalyst/jackson-kotlin-test).

https://github.com/FasterXML/jackson-module-kotlin/issues/545