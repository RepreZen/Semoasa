# SEMOASA - Specification Extension Metadata for OpenAPI (DRAFT)
## Overview
The OpenAPI Specification, formerly known as the Swagger Specification, is a standard, machine-readable format for REST-style API definitions and documentation. Swagger 2.0 introduced _vendor extensions_, which allow certain objects to have properties named with an `x-` prefix, with arbitrary or independently specified property values. Vendor extensions were later renamed to [specification extensions](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#specificationExtensions) in OpenAPI 2.0 and 3.0.

While specification extensions are [widely used](https://github.com/Mermade/openapi-specification-extensions/blob/master/extensions/combined.tsv), there is no standardized way to define their syntax, expected values, and allowed usage context. This makes it much more difficult for OpenAPI tools to support specification extensions with content assist, validation, and other features that those tools typically provide for standard OpenAPI language constructs. 

For example, an OpenAPI editor will usually have design-time knowledge of the OpenAPI specification, and _may_ make use of a JSON Schema that encodes some of this knowledge. Based on this knowledge, it can provide context-sensitive editing assistance and validation. But if the user adds extended properties with the `x-` prefix, the trail goes cold. Unless the editor has been designed with explicit prior knowledge of that specification extension, it can't tell the user what the extended property means, what kinds of values are expected, and whether that extension is even meaningful in this context. 

SEMOASA is a machine-readable format for specification extensions that provides a base level of structural and descriptive information, along with optional links to external resources for human-readable documentation. Like OpenAPI itself, specification extensions may have certain constraints and semantics that aren't captured in a schema or metadata descriptor; but the information carried in SEMOASA is sufficient to allow OpenAPI tools to provide a user experience much closer to what they provide for standard OpenAPI syntax.

## Use Cases

SEMOASA is designed with a few representative use cases in mind:

* **Tool Functions**
    * *Content Assist* - Providing in-place code assist, or "Intellisense", including proposals for specification extension properties intended for use in the current context. 
    * *Tool Tips* - Extension property descriptions and/or formatted documentation, visible when hovering over an actual or proposed usage of the extension property. 
    * *Validation* - Checking usage of extension properties against the provided extension descriptor, flagging errors or warnings where an extended property appears in an unexpected context, or has a value that does not conform to the specified schema. 
* **Registry Patterns** 
   * *Direct Registration* - Consumers of specification extension metadata may register an individual SEMOASA document, directly from its canonical location. 
   * *Directory Catalog* - A publisher may provide a list of known OpenAPI specification extensions, referring back to the canonical source location for the actual extension metadata.  A consumer of a Directory Catalog would need traverse references to those locations to retrieve the metadata. 
   * *Aggregation Catalog* - A publisher may provide a collection of known OpenAPI specification extensions, including the full extension metadata. This allows a consumer to ingest a full set of metadata without requiring additional round trips. In this case, it should still be possible to include the canonical location, so the client can refresh these definitions as needed.  

## Design Goals
SEMOASA aims to meet the following goals, to the extent practical:
* *Consistency with OpenAPI*, so developers familiar with current OpenAPI concepts, terminology and style will be familiar with corresponding elements of SEMOASA.
* *Common Technology Stack with OpenAPI*, so that essential functions like YAML editing, schema validation, and JSON references can be performed by the same components used with OpenAPI tool implementations. 
* *Support for OpenAPI 2.0 and later*, to ensure relevancy with current and future usage.
* *Provider-Neutrality*, not being significantly influenced by a particular set of tools or OpenAPI specification extension providers.



## Structure
## Examples
