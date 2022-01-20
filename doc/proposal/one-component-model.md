# One Component Model Specification

This document specifies the *One Component Model (OCM)*.

## Introduction

### Scope

Operating software installations both for cloud and on-premises covers many aspects:

- how, when and where are the technical artefacts created
- how are technical artefacts stored and accessed
- which technical artefacts are to be deployed
- how is the configuration managed
- when are technical artefacts deployed
- where and how are those artefacts deployed
- which other software installations are required and how are they deployed and accessed

The overall problem domain has a complexity that make it challenging to be solved as a whole.
However, the problem domain can be divided into two disjoint phases:

- production of technical artefacts
- deployment and lifecycle management of technical artefacts

The produced artefacts must be stored somewhere such that they can be accessed and collected for the deployment.
The OCM defines a standard to describe which technical artefacts belong to a software installation and how to 
access them. This provides a clear interface between the production and the deployment/lifecycle management phase.

The OCM does not make any assumptions about the 
 
- kinds of technical artefacts (e.g. docker images, helm chart, binaries etc.)
- technology how to store and access technical artefacts (e.g. as OCI artefacts in an OCI registry)

Implementations of the OCM must define exactly those aspects. 

**ToDo: Reference to our OCI based realization**

### Motivation Example

Usually complex software products are divided into logical units, which are called components in this specification.
For example a software product might consist of three components, a frontend, a backend and some monitoring stack.
Of course the software product itself could be seen as a component comprising the other three components.

As a result of the development phase, versions of the components are created. 

A version of a component consists of a set of technical artefacts, e.g. docker images, helm charts, binaries, 
configuration data etc. Such artefacts are called resources in this specification.

The OCM introduces a so called *Component Descriptor*, to define the resources belonging to a particular component 
version and how these resources could be accessed. 

For the three components of our example software product, one component descriptor exists for every component version,
e.g. three component descriptors for the three versions of the frontend, six for the six versions of the backend etc.

Not all component version combinations of frontend, backend and monitoring result are compatible. 
In order to define reasonable version combinations for our software product we could use another feature of 
the *Component Descriptor*, which allows the definition of dependencies to other component versions. For our example
we could introduce a component for the overall product. Different versions of this product component are again
described by a *Component Descriptor*, which contain dependencies to particular *Component Descriptors* for the 
frontend, backend and monitoring. 

Every *Component Descriptor* has a name and a version. Dependencies to other *Component Descriptors* are expressed
by their name and version.

This is only an example how to describe a product with OCM as a component with one *Component Descriptor* 
with dependencies to other *Component Descriptors*, which itself could have dependencies and so on. 
You are not restricted to this approach, i.e. you could still just maintain a list of component version combinations which 
build a valid product release. But OCM provides you a simple approach to specify what belongs to a product version. 
Starting with the *Component Descriptor* for a product version and following the component dependencies, you could 
collect all artefacts, which belong to the product version. 

*Component Descriptors* are stored in *Component Repositories*. A *Component Repository* must provide a possibility 
to access every stored *Component Descriptor* only by its name and version. Another requirement to a 
*Component Repository* is, that uploading a *Component Descriptors* fails if it has a dependency to another
*Component Descriptor* which is not contained in the *Component Repository*.

The concept of referencing other *Component Descriptors* only by their name and version allows the transport of
the *Component Descriptors* from one *Component Repository* to another without the need to change component
references in a *Component Descriptor* because the names and versions remain stable. Resources (technical artefacts)
referenced by the *Component Descriptors* could either remain untouched or also transported to a new location. In the
latter case, the references in the transported *Component Descriptors* have to be updated to their new locations.

**Todo: Perhaps some small example image to make this more clear?**

### Application Areas

OCM specifies a well-defined way to define components and resources, belonging to a particular version of a software 
product, and how to access these. Though the following application areas are out of scope for OCM, it provides a 
re-usable, and technology-agnostic interface for compliance checks, security scanning, code signing, transport, deployment
or other lifecycle-management aspects. 