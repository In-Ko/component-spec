# One Component Model Specification

This document specifies the *One Component Model (OCM)*.

## Introduction

### Scope

Operating Software installations both for Cloud and on-premises covers many aspects:

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

The produced artefacts must be stored somewhere such that it can be accessed and collected for the deployment.
The OCM defines a standard to describe which technical artefacts belong to a software installation and how to 
access them. This provides a clear interface between the production and the deployment/lifecycle management phase.

The OCM does not make any assumptions about the 
 
- kinds of technical artefacts (e.g. docker images, helm chart, binaries etc.)
- technology how to store and access technical artefacts (e.g. as OCI artefacts in an OCI registry)

Implementations of the OCM must define exactly those aspects. 

**ToDo: Reference to our OCI based realization**

### Motivation Example

Usually complex software products are divided into smaller parts, which are called components in this specification.
For example a software product might consist of three components, a front end, a backend and some monitoring stack.

As as result of the development phase versions of the components are created. 

A version of a component consists of a set of technical artefacts, e.g. docker images, helm charts, binaries, 
configuration data etc. Such artefacts are called resources in this specification.

The OCM introduces a so called *Component Descriptor*, to define the resources belonging to a particular component 
version and how these resources could be accessed. 

For the three components of our example software product, one component descriptor exists for every component version,
e.g. three component descriptors for the three versions of the frontend, six for the six versions of the backend etc.

Of course not all component version combinations of frontend, backend and monitoring result are compatible. 
In order to define reasonable versions combinations for our software product we could use another feature of 
the *Component Descriptor*, which allows the definition of dependencies to other component versions. For our example
we could introduce a component for the overall product. Different versions of this product component are again
described by a *Component Descriptor*, which contains dependencies to particular *Component Descriptors* for the 
frontend, backend and monitoring. Of course this is only an example how to describe a product as a components with 
*Component Descriptors* but it is not mandatory to do so. 

Every *Component Descriptors* has a name and a version. Dependencies to other *Component Descriptors* are expressed
by their name and version.

*Component Descriptors* are stored in *Component Repositories*. A *Component Repository* must provide a possibility 
to access every stored *Component Descriptor* only by its name and version. Another requirement to a 
*Component Repository* is, that uploading a *Component Descriptors* fails if it has a dependency to another
*Component Descriptor* which is not contained in the *Component Repository*.

The concept of referencing other *Component Descriptors* only by their name and version allows the transport of
the *Component Descriptors* from one *Component Repository* to another without the need to change component
references in a *Component Descriptor* because the name and version of remain stable.

**Todo: Perhaps some small example image to make this more clear?** 



Each of the artefacts declared in a `Component Descriptor` has an `identity`, an `access`
description and a `type`. `Artefact identities` can be used to reference an artefact in
the context of the declaring component descriptor. The `Artefact type` defines how the
artefact is to be interpreted. The `access Description` defines from where the artefact
can be retrieved.

When replicating component descriptors and their artefacts, the artefacts' `access` descriptions
may be changed. However, `artefact types` and `artefact identities` always remain unchanged.

The CNUDIE Component Model can roughly be compared to a filesystem "living" in an arbitrary
storage (e.g. a blobstore, oci-registry, archive, ..).

Starting from an arbitrary component repository used as root repository it is
possible to access any artefact described by the component model as long as the
dedicated component version has been imported into this repository. The
dedicated component repository together with the component version identity
allows to access the component descriptor which then is used to determine the
access for dedicated artefacts given by their local identity. This way the
component respository acts as composed filesystem with fixed top level folders,
the components and their versions, followed by the artefact level, which then
may point into any another (kind of) repository. Sub folders can be described
by component version references described by component descriptors, which again
feature a local identity in the describing component descriptor.


Scope Definition
----------------

The CNUDIE Component model intends to solve the problem of addressing,
identifying, and accessing artefacts for software components, relative to an
arbitrary component repository. By that, it also enables the transport of
software components between component repositories.

Through the standardisation of structure and `access` to artefacts, it can serve as an
interface to any operations that need to interact with the content. This allows
for tools operating against this interface to be implemented in a re-usable, and
technology-agnostic manner (examples being transport, compliance and security
scanning, codesigning, ..).

Higherlevel functionality, such as deployment, or lifecycle-management aspects are
out of the scope the CNUDIE Component Model targets. Data for such aspects (for example
a deployment blueprint) can, however, be described by the CNUDIE Component Model
as dedicated typed artefacts. This is equivalent to e.g. adding a `Makefile`
into a filesystem. The filesystem does not "know" about the semantics of the
`Makefile` (including, e.g. declared dependencies towards other files).