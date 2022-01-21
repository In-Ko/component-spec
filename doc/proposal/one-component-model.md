# Open Component Model Specification

This document specifies the *Open Component Model (OCM)*.

## Introduction

### Scope

Operating software installations/products both for cloud and on-premises covers many aspects:

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

Though the following application areas are out of scope for OCM, it provides a uniform interface for 
compliance checks, security scanning, code signing, transport, deployment or other lifecycle-management aspects. 
If software installations are described using OCM, e.g. a scanning tool could use this to collect all technical 
artefacts it needs to check. If the technical resources of different software installations are described with different 
formalisms, such tools must provide interfaces and implementations for all if them. 

The problem becomes even harder if a software installations is build of different parts/components, each described with 
another formalism. OCM allows a uniform definition of such dependencies such that one consistent description of
for a software installation is available.

The OCM does not make any assumptions about the (**Todo: Is this really the case?**)
 
- kinds of technical artefacts (e.g. docker images, helm chart, binaries etc.) 
- technology how to store and access technical artefacts (e.g. as OCI artefacts in an OCI registry)

Implementations of the OCM must define exactly those aspects. 

**ToDo: Reference to our OCI based realization**

### Motivation Example

Usually complex software products are divided into logical units, which are called components in this specification.
For example, a software product might consist of three components, a frontend, a backend and some monitoring stack.
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

## Component Descriptor Specification

*Component Descriptors* are the central concept of OCM. A *Component Descriptor* describes what belongs to a particular 
version of a software component  and how to access it. This includes:

- resources, i.e. technical artefacts like binaries, docker images, ...
- sources like code in github
- dependencies to other software component versions

### Component Descriptor Format Specification

A *Component Descriptor* is a [YAML](https://yaml.org/) or [JSON](https://www.json.org/json-en.html) document 
according to this [schema](../../language-independent/component-descriptor-v2-schema.yaml).

In serialised form, Component Descriptors MUST be UTF-8-encoded. Either YAML, or JSON may be used. If YAML is used 
as serialisation format, only the subset of features defined by JSON must be used, thus allowing conversion to a 
JSON representation.

YAML is recommended as preferred serialisation format.

YAML permits the usage of comments, and allows different formatting options. None of those are by contract part of a 
*Component Descriptor*, thus implementations may arbitrarily choose to retain or not retain comments or formatting 
options.

The order of attributes is insignificant, and MUST NOT be relied upon.

The order of elements in sequences MAY be significant and MUST be retained in cases where it is significant.

### Schema Version

A *Component Descriptor* document consist of two top level elements: *meta*, *component*

The *meta* element contains the schema version of the *Component Descriptor* specification. This document defines
schema version *v2*.

```
meta:
  - schemaVersion: "v2"
component:
  ...
```


### Name and Version

Every *Component Descriptor* has a name (*component.name*) and version (*component.version), also called component 
name and version. Name and version are the identifier for a *Component Descriptor* and the component version described
by it.

```
meta:
  - schemaVersion: "v2"
component:
  name: ...
  version: ...
```

Component names reside in a global namespace. To avoid name conflicts component names MUST start with a valid domain 
name (as specified by RFC-1034, RFC-1035) with an optional URL path suffix (as specified by RFC-1738).

If no URL path suffix is specified, the domain MUST be possessed by the component proprietor. If a URL path suffix is 
specified, the namespace started by the concatenation of domain and URL path suffix MUST be possessed by the component 
proprietor.

The component name SHOULD reference a location where the component’s resources (typically source code, and/or 
documentation) are hosted. An example and recommended practise is using GitHub repository names for components on 
GitHub like *github.com/gardener/gardener*.

Component versions refer to specific snapshots of a component. A common scenario being the release of a component.
Component versions MUST adhere to a loosened variant of [Semver 2.0.0](https://semver.org/).

Different to strict semver 2.0.0, component versions MAY:

- have an optional v prefix
- omit the third component (patch-level); if omitted, path-level is implied to equal 0

### Sources and Resources

Components versions are typically built from sources, maintained in source code management systems, 
and transformed into resources (for example by a build), which are used at installation or runtime for the product.

Resources are “local” if they are derived from a source declared by the same component. Resources are “external” if
they are not derived form a source declared by the same component, e.g. a docker image for a monitoring stack
developed in another component. 

Each *Component Descriptor* contains a field with references to the used sources and a field with references 
to the used resources. 

Example for sources: 

```
...
component:
  sources:
  - name: extension-networking-calico
    version: v1.19.4
    type: git
    access:
      commit: e01326928b6f9825dba9fa530b8d4917f93194b0
      ref: refs/tags/v1.19.4
      repoUrl: github.com/gardener/gardener-extension-networking-calico
      type: github
      ...
```

Every source entry has a name and version field. Furthermore, it has a type field, specifying the nature of the 
source code management system, and a type specific access section.





