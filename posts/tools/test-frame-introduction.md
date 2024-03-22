---
title: Make your testing easier with Test-Frame
menu_order: 1
post_status: publish
post_excerpt: Introduction to Test-Frame - what it is, why you should use it
featured_image: _images/test-frame-introduction.jpg
author: lkral
taxonomy:
    category:
        - tools
---

## Testing of applications running on Kubernetes

Today, there are multiple popular languages that are used for writing Kube-native applications.
Most popular are Java and Go, mainly in development of application based on [operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/).
Additionally, there are multiple libraries for both of the aforementioned languages, that make the work 
with Kubernetes API much easier.
One of them is [Fabric8's](https://fabric8.io/) [kubernetes-client](https://github.com/fabric8io/kubernetes-client), which allows developers 
direct communication with Kubernetes API.
Because of this library (and also many other like [JUnit5](https://junit.org/junit5/)), 
we decided to write the test-suite in Java, which is also the language our
application is written in.

## Creating a test-suite

At the beginning, you start with observing what is needed during each phase of the test case.
You can start with the setup needed before the tests execution.
Then, during the test case, you can create and delete the Kubernetes resources, do operations 
that your application allows, check logs of the Pod, validate results, clean-up the environment after 
the test case.

Having a clean-up phase in every test case is sufficient when you have just few test cases.
But once you are writing more and more tests, it starts to be boring and repetitive when you
have to clean the resources at the end of each test.

To make your life easier, you create a mechanism that will store the resources created in the test
and once the test is completed, it will clear everything based on LIFO mechanism.

Great, what about the situation when the test fails?
You would have to simulate the same environment and situation in which it failed.
But sometimes, it is not that simple to do reproduce the exact behavior.
Because of this, you create another mechanism for collecting logs, description of resources, and many more, 
that will be executed in case of the test failure.

## More repositories, more redundancy

In case that we have just one repository where all the testing is done, everything is fine.
But once we want to extend the product and add new components that will also need system testing, 
we start a fight with redundancy.
And, one day, you will find yourself copying the testing related classes across multiple repositories,
mixing things and always trying to refactor the code without success and without "the light at the end of a tunnel" that
there will be an end to this.
At least, that is the situation in which we ended up in.

## Test-Frame as the solution

### What it provides?

### How I can use it?

## Conclusion