---
title: Data Generator Library for Streaming 0.2.0
menu_order: 1
post_status: publish
post_date: 2024-07-25 09:00:00
post_excerpt: Release of Data-Generator 0.2.0 is out!
featured_image: _images/data-generator-0_2_0-release.jpg
author: jstejskal
taxonomy:
  category:
    - releases
    - data-generator
---

## Data-Generator - What and Why?
In the past few weeks, I started thinking about improving the data we use in our long-running test environment. 
Since we are using Strimzi and Kafka, along with test clients for load from Strimzi org, I thought about making a small enhancement. 
Currently, the clients only generate the same string. Not great, not terrible—just enough to monitor the load. 
But what if you want to do more with the data? Not just check if the storage can handle it, but maybe add some tools for stream processing? 
Simple text messages are kinda useless, especially when they are always the same!

So, our team did some digging around and found the [jr tool](https://github.com/ugol/jr). 
This project is quite cool, but we wanted something we could integrate into our existing code. 
That was when I started thinking about our own generator based on the [data-faker](https://www.datafaker.net/) library.

### Data-Faker

Data-faker is a great library that you can use to fake any kind of data in your Java applications. 
Since we mostly use Java, this library was our first choice for reasonably generating random data.

The usage is pretty simple. Here's an example from the quick start guide:
```java
import net.datafaker.Faker;

Faker faker = new Faker();

String name = faker.name().fullName(); // Miss Samanta Schmidt
String firstName = faker.name().firstName(); // Emory
String lastName = faker.name().lastName(); // Barton

String streetAddress = faker.address().streetAddress(); // 60018 Sawayn Brooks Suite 449
```

Pretty handy, right? You can easily extend the input data for data-faker, which can significantly help with the data you want to generate.

For example, you can specify your own list of options in the code or extend the data through a YAML file.

```java
FAKER.options().option("active", "inactive", "error");
```

### How We Are Using It
You can easily create your own data generator based on your needs. Since we heavily use the [test-clients](https://github.com/strimzi/test-clients) tool developed in Strimzi org, we decided to enhance it with functionality to generate more than just simple text messages.

As part of the data-generator, we put together a few templates to solve this problem. 
Each template is based on an Apache Avro schema. Maven generates Java classes from these schemas, and we simply fill these classes with random or pseudo-random data based on our needs. 
Then we use the generator itself in our client implementation, and voila—we have a reasonable message for our environment!

As an example, consider the following schema (already implemented as part of the data-generator):
```json
{
  "type": "record",
  "name": "StarGate",
  "namespace": "io.skodjob.datagenerator.models.stargate",
  "fields": [
    {
      "name": "character_name",
      "type": "string"
    },
    {
      "name": "source_planet",
      "type": "string"
    },
    {
      "name": "target_planet",
      "type": "string"
    },
    {
      "name": "quote",
      "type": "string"
    },
    {
      "name": "duration",
      "type": "int"
    },
    {
      "name": "duration_unit",
      "type": "string"
    },
    {
      "name": "distance",
      "type": "int"
    },
    {
      "name": "distance_unit",
      "type": "string"
    }
  ]
}
```
Now, just fill the template with data:
```java
public static String generateData() {
    StarGate starGate = new StarGate();
    starGate.setCharacterName(FAKER.stargate().characters());
    starGate.setSourcePlanet(FAKER.stargate().planets());
    starGate.setTargetPlanet(FAKER.stargate().planets());
    starGate.setQuote(FAKER.stargate().quotes());
    starGate.setDuration(FAKER.number().numberBetween(1, 50));
    starGate.setDurationUnit("seconds");
    starGate.setDistance(FAKER.number().numberBetween(20000, 999999));
    starGate.setDistanceUnit("light_year");

    return starGate.toString();
}
```

The final message will look like this:
```json
{
  "character_name": "Selmak",
  "source_planet": "Langara",
  "target_planet": "Earth",
  "quote": "What is an Oprah?",
  "duration": 4,
  "duration_unit": "seconds",
  "distance": 999043,
  "distance_unit": "light_year"
}
```

That was just a simple example, but it is enough for demonstration. Currently, we have 6 templates implemented:
- PaymentFiat
- Flights
- Payroll
- IotDevice
- StarWars
- StarGate

We plan to extend it based on our needs for reasonable data in our test environment!
Also, as a bonus you have also Avro schema that you can use in your registry to validate a messages against it.

## Conclusion
Our current version is 0.2.0, and we are using it in Strimzi's [test-clients](https://github.com/strimzi/test-clients) implementation. 
The implementation is more or less for our needs, but in case you haven't heard about the data-faker library, it's something very useful for everyone! 
If you need Kafka test clients with pre-defined data or your own, Strimzi's test-clients are a good start as well!
