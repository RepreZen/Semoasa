# SEMOA - Specification Extension Metadata for OpenAPI (DRAFT)
## Overview
The OpenAPI Specification, formerly known as the Swagger Specification, is a standard, machine-readable format for REST-style API definitions and documentation. Swagger 2.0 introduced _vendor extensions_, which allow certain objects to have properties named with an `x-` prefix, with arbitrary or independently specified property values. Vendor extensions were later renamed to [specification extensions](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#specificationExtensions) in OpenAPI 2.0 and 3.0.

While specification extensions are widely used, there is no standardized way to define their syntax, expected values, and allowed usage context. This makes it much more difficult for OpenAPI tools to support specification extensions with content assist, validation, and other features that those tools typically provide for standard OpenAPI language constructs. 

For example, an OpenAPI editor will usually have design-time knowledge of the OpenAPI specification, and _may_ make use of a JSON Schema that encodes some of this knowledge. Based on this knowledge, it can provide context-sensitive editing assistance and validation. But if the user adds extended properties with the `x-` prefix, the trail goes cold. Unless the editor has been designed with explicit prior knowledge of that specification extension, it can't tell the user what the extended property means, what kinds of values are expected, and whether that extension is even meaningful in this context. 

SEMOA is a machine-readable format for specification extensions that provides a base level of structural and descriptive information, along with optional links to external resources for human-readable documentation. Like OpenAPI itself, specification extensions may have certain constraints and semantics that aren't captured in a schema or metadata descriptor; but the information carried in SEMOA is sufficient to allows OpenAPI tools to provide a user experience much closer to what they provide for standard OpenAPI syntax.

## Use Cases
## Structure
## Examples
