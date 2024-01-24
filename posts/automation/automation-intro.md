---

title: What does automation-hub mean?
menu_order: 1
post_status: publish
post_excerpt: If you are interested in cool technologies for DevOps and automation like Ansible, Tekton, or ArgoCD, check out this introduction to our project.
featured_image: _images/automation.jpg
author: dkornel
taxonomy:
    category:
        - automation

---

## What is the automation-hub
Simplified answer is that [automation-hub](https://github.com/skodjob/automation-hub) is a collection of [ansible](https://www.ansible.com/) automation scrips, [kubernetes](https://kubernetes.io/)
deployment and configuration files, [Tekton](https://tekton.dev/) pipelines an [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) applications and application-sets. Let's talk about used technologies,
why and how we use it for our work.

### Kubernetes
Kubernetes is known as the best way to orchestrate containerized applications. It provides scalability, reliability and many more features for applications running on top of the kubernetes.
In our case we use enterprise implementation of kubernetes called [Openshift](https://www.redhat.com/en/technologies/cloud-computing/openshift/container-platform). That was just brief explanation
that kubernetes is the platform for running containerized applications, and now the question "Why we use it"? Answer is simple
"We use kubernetes, because we work as **QA** and applications which we put under test are targeted to kubernetes."

### Ansible
Do you know the situations when you have many automation scripts written in bash or powershell and you start loosing mind
in a bunch of `$` or `sed` expressions? We know how frustrating can be debugging shell script or running them on the remote machine.
But there is a better way which is `Ansible`. Ansible is a declarative way how to write automation scrips in yaml format, which also allows you to use
libraries with already implemented automation for many components, so you do not need re-invent the wheel. Because initial installation path
of our environment requires a lot of steps, Ansible it the correct way to deduplicate steps, create a templates in `jinja` and drive
execution based on tags. We are able to run Ansible playbook on provided kubernetes cluster, and it deploys the whole
testing infrastructure including multi-cluster environment with all needed operators and tools (we will talk about all of these stuff in upcoming posts). 

### Tekton
There are many ways where to run CI/CD pipelines. You surely know systems like Gitlab-CI, Jenkins or TeamCity. Tekton is also CI/CD system,
but it has an advantage which is really important for us, it is `cloud-native`. That means it is implemented to run on top of kubernetes, and it is driven by resources written in yaml.
Tekton pipeline resource allows you to split steps you want to run into different container images, it can save you
resources needed for running pipeline, and also you do not need to build large container image with all required tools installed. Let me
present it on simple example. Imagine pipeline, which does api request using `curl` and then run `maven`, the first step can be done in a
simple small container just with curl installed and the maven one step can run in different container with java and maven installed.

We use Tekton CI/CD mainly for continuous check of deployed environment and system under test, also for trigger kubernetes cluster updates and many more
day to day operational stuff.

### ArgoCD
All previous technologies were mostly about platform and DevOps practices, but now let's move to another buzzword `GitOps`.
ArgoCD is GitOps tool for deployment kubernetes applications from declarations stored in SCM. Basically if someone put changes
of application configuration into git repository, ArgoCD gets this changes and apply them to target kubernetes cluster. All our
systems under tests are stored in [deployment-hub](https://github.com/skodjob/deployment-hub) repository and ArgoCD takes care about distributing
them to multi-cluster environment.
