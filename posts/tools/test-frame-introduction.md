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
The most popular are Java and Go, mainly in the development of applications based on [operator pattern](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/).
Additionally, there are multiple libraries for both of the aforementioned languages, which make the work
with Kubernetes API much easier.
One of them is [Fabric8's kubernetes-client](https://github.com/fabric8io/kubernetes-client), which allows developers
direct communication with Kubernetes API.
Because of this library (and also many others like [JUnit5](https://junit.org/junit5/)),
we decided to write the test suite in Java, which is also the language that our
application is written in.

## Creating a test-suite

In the beginning, you start with observing what is needed during each phase of the test case.
You can start with the setup needed before the test execution.
Then, during the test case, you can create and delete the Kubernetes resources, do operations
that your application allows, check logs of the Pod, validate results, and clean-up the environment after
the test case.

Having a clean-up phase in every test case is sufficient when you have just a few test cases.
But once you are writing more and more tests, it starts to be boring and repetitive when you
have to clean the resources at the end of each test.

To make your life easier, you create a mechanism that will store the resources created in the test
and once the test is completed, it will clear everything based on the LIFO mechanism.

Great, what about the situation when the test fails?
You would have to simulate the same environment and situation in which it failed.
But sometimes, it is not that simple to reproduce the exact behavior.
Because of this, you create another mechanism for collecting logs, description of resources, and many more,
that will be executed in case of the test failure.

All of these will be mostly stored in the repository where you are writing the tests, which makes sense
and it is the best possible solution at that moment.

## More repositories, more redundancy

In case we have just one repository where all the testing is done, everything is fine.
But once we want to extend the product and add new components that will also need system testing,
we start a fight with redundancy.
And, one day, you will find yourself copying the testing-related classes across multiple repositories,
mixing things and always trying to refactor the code without success and without "the light at the end of the tunnel" that
there will be an end to this.
At least, that is the situation in which we ended up.

Yes, you can use the test classes from the main repository as a Maven dependency, in case you are releasing it together
with other modules to the Maven Central (or anywhere else).
However, there are situations when you cannot do that - one of the examples is cyclic dependency.

Let's say that you have the main repository, containing the code for an operator, and you have a few other repositories for the components that
are managed by the operator.
Each of the components has a different release cadence, so having them in the operator's repository is not the correct approach.
In this particular case, if you would use the "system test" dependency from the operator's repository inside the component's repository (and the component
is used inside the operator's repository), then you create a cyclic dependency, where the component waits for the release of the operator
with new things in the test classes, and the operator waits for the component to be released, so it can be included in the controller's code.

Or, you can have one repository for the test logic, that will be shared across multiple repositories, and thanks to that you will remove
the cyclic dependency.
Nevertheless, with a new separate repository, there are different challenges.
You need to think about how well it will be maintained - do you and your team have time to actively contribute to this repository and release a new
version of it regularly, without breaking the logic of the test suites in each of the repositories?
How frequent will be the releases?
What about the situation when you work on different repositories outside of the current organization and this module cannot be imported?

## Test-Frame as the solution

Because Skodjob consists of engineers working on different projects, but with the same passion for testing the integration between the components,
we were finding a proper solution for sharing the code between organizations, that would help us with removing redundancy and duplicated code, which
was anyway present throughout the repositories.
And because we had the same questions I mentioned in the previous section, we decided to create a library called [Test-Frame](https://github.com/skodjob/test-frame/).
The library aims to provide basic functionality for testing applications running on Kubernetes.
It also tries to make the overall process as easy as possible.

### What it provides?

The core of the Test-Frame is `KubeResourceManager` which is responsible for the management of resources during the test phases.
For its functioning, we need to pass Fabric8-based objects, that are extending the `HasMetadata` class, to it.
Depending on the `ExtensionContext`, `KubeResourceManager` creates a new stack for each context, where it stores all resources created in the test case.
After the test is finished (no matter if it failed or passed), `KubeResourceManager` deletes the resources in reverse order, following the LIFO
method.
This removes the need for "manual" clean-up after each test case and overall management of the resources.

Besides that, Test-Frame provides classes for easier handling of basic Kubernetes resources, such as Secret, Service, NetworkPolicy, and
many more.
It also has built-in `KubeClient`, which creates a basic encapsulation around the Fabric8's client, together with the possibility to
specify `KUBE_CONTEXT` for the cluster where the tests should be executed.

The library is still under development, we are planning to add more functionality like a log collector (which will collect also YAMLs of resources, Pod description, and more)
or metrics collector, for easier gathering metrics from the pre-defined endpoints.

### How I can use it?

During the development, the Test-Frame library is being pushed to GitHub's Maven repository.
So in case you would like to try the Test-Frame in your test suite before it is released, you can do that by including
the GitHub's Maven repository and the Test-Frame dependency in your `pom.xml`:

```xml
    <repositories>
        <repository>
            <id>test-frame</id>
            <name>GitHub Apache Maven Packages</name>
            <url>https://maven.pkg.github.com/skodjob/test-frame</url>
        </repository>
    </repositories>
    <dependencies>
        <dependency>
            <groupId>io.skodjob</groupId>
            <artifactId>test-frame-common</artifactId>
            <version>0.1.0-SNAPSHOT</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
```
where the `test-frame-common` module contains the core functionality - like the `KubeResourceManager`.
Additionally, there are more modules that you can use and explore in our [GitHub repository](https://github.com/skodjob/test-frame).

Because we are pulling a dependency from the GitHub Maven repository, we need to provide an additional configuration in the `settings.xml`:

```xml
<settings>
    <servers>
        <server>
            <id>test-frame</id>
            <username>x-access-token</username>
            <password>${env.GITHUB_TOKEN}</password>
        </server>
    </servers>
</settings>
```

Where the `id` of the server has to match with the `id` of the repository in the `pom.xml`.

Once you are all set, you can try to experiment with the library in the tests by annotating the test classes with
`@ResourceManager` annotation, determining that the `KubeResourceManager` should manage the resources created in the tests.

```java
@ResourceManager
class NamespaceCreationST {
    
    @Test
    void testThatNamespaceExists() {
        KubeResourceManager.getInstance().createResourceWithWait(
                new NamespaceBuilder().withNewMetadata().withName("test").endMetadata().build());
        assertNotNull(KubeResourceManager.getKubeCmdClient().get("namespace", "test"));
    }
}
```

In this example, the Namespace with the name `test` is created in the test case `testThatNamespaceExists`, and after the test
is finished, it is also deleted by the `KubeResourceManager`.
This will ensure that the Namespace will not affect any other tests, which will start again in a clean environment.

## Conclusion

The process of writing tests with all kinds of helper classes and methods can be a repetitive task filled with duplicates and
redundancy when it comes to a project containing multiple repositories that need testing.
But Test-Frame, a library providing common helper methods for work with tests and objects running on Kubernetes, is here
to help you with it.

Let us know how you like the library or what would you like to see next.
Your feedback is welcome!