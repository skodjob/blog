---
title: Data Generator library for streaming 0.2.0
menu_order: 1
post_status: publish
post_date: 2024-07-23 09:00:00
post_excerpt: Release of Data-Generator 0.2.0 is out!
featured_image: _images/data-generator-0_2_0-release.jpg
author: kornys
taxonomy:
  category:
    - releases
    - data-generator
---

## Data-Generator - for what? Why?
In the past weeks I started thinking about improvement of data we are using in our long-running test environment. 
As we are using Strimzi and Kafka here, and test-clients for load from Strimzi org, it makes me thing about a small improvement.
Currently, the clients only generate same String.
Not great, not terrible just to monitor load, but what if you want to do something more with the data?
Not just see if the storage is handling it, but maybe add some other tools for stream processing? 
With just simple text messages it is kinda useless. 
Especially when the messages are still the same!

So in our team we did some digging around and found [jr tool](https://github.com/ugol/jr).
This project is quite cool, but we wanted something we could integrate into our existing code.
That was the time when I was string thinking about our own generator based on [data-faker](https://www.datafaker.net/) library.

### Data-Faker

Data-faker is a great library that you can use to fake any kind of data in your Java applications.
Because we are using mostly Java for our stuff, this library was the first choice how to reasonably generate random data.

The usage is pretty simple. Let's show the example from the quick start guide:
```Java
import net.datafaker.Faker;

Faker faker = new Faker();

String name = faker.name().fullName(); // Miss Samanta Schmidt
String firstName = faker.name().firstName(); // Emory
String lastName = faker.name().lastName(); // Barton

String streetAddress = faker.address().streetAddress(); // 60018 Sawayn Brooks Suite 449
```

Pretty handy right? 
You can easily extend the input data for data-faker which could significantly help you with the data you want to generate.

For example specify your own list of option sin the code or also extend the data through yaml file.

```Java
FAKER.options().option("active", "inactive", "error");
```

### How we are using it
You can easily create your own data-generator based on your needs.  
Because we are heavily using [test-clients](https://github.com/strimzi/test-clients) tool we developed in Strimzi org, we decided to enhance them with functionality to generate not just a simple text messages.

As part of data-generator we put together few templates that solves this problem.
Each template is based on Apache Avro schema.
Maven generates Java classes from these schemas and we simply fill-up the classes with random or pseudo-random data based on our needs.
Then we use the generator itself in our client implementation and vioala - we have reasonable message for our environment!

As an example let's consider the following schema (it is implemented as part of data-generator already):
```avroschema
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
Now just fill-up the template with data:
```Java
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

That was just a simple example, but it is enough for demonstration. 
Currently, we have 6 templates implemented:
- PaymentFiat
- Flights
- Payroll
- IotDevice
- StarWars
- StarGate

but we plan to extend it based on our needs for reasonable data in our test environment!

## Conclusion
Our current version is 0.2.0, and we are using it in Strimzi's test-clients implementation. 
The implementation is more or less for our needs, but in case you didn't hear about data-faker library, that's something very useful for everyone!
And in case you want some Kafka test-clients with pre-defined data or your own, test-clients from Strimzi are good start as well!

