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
* **Consistency with OpenAPI**, so developers familiar with current OpenAPI concepts, terminology and style will be familiar with corresponding elements of SEMOASA.
* **Common Technology Stack with OpenAPI**, so that essential functions like YAML editing, schema validation, and JSON references can be performed by the same components used with OpenAPI tool implementations. 
* **Support for OpenAPI 2.0 and later**, to ensure relevancy with current and future usage.
* **Provider-Neutrality**, not being significantly influenced by a particular set of tools or OpenAPI specification extension providers.
* **Extension Identity**, allowing consumers to disambiguate duplicate names. Since extension providers are not required to register unique property names in a central location, name clashes can occur. To meet this requirement, SEMOASA introduces _namespace_ to define a set of extension properties under coordinated management.

## Structure

* **Extensions Object** - an object defining one or more extensions, organized by namespace. 
    * **openapiExtensionFormat** - a version string indicating the format used to describe the extensions. Allows tools to support multiple versions of specification extensions. Each revision of the specification will prescribe a specific version to use. Should adhere to semantic versioning, allowing tools to support a given major or minor version. 
    * **\[namespace\]** (patterned field, type: Namespace Object or reference) - a namespace encompassing a set of managed Specification Extensions. Within a given namespace, management of Specification Extensions is expected to be quasi-coordinated, at least to the extent of ensuring unique extension names. As a convention, namespaces should generally use reverse-DNS notation, with additional segments as needed, e.g. `com.adobe.experience.public`. (Pattern for this set of properties can enforce that convention, and thus exclude statically defined properties like openapiExtensionFormat.) May contain a set of Extension Descriptors, or a JSON Reference to a Namespace Object, to support the _Directory Catalog_ use case.
        * **\[extensionPropertyName\]** (patterned field, type: Extension Object or reference)- name of the specification extension property. must start with "x-" as per the OAS specification. We may want to support wildcards or template parameters in the name. May contain an Extension Object or a JSON Reference thereto, supporting the _Directory Catalog_ use case (one level deeper than namespace). 
            * **summary** - a plain-text summary of the purpose of the extension. 
            * **description** - markdown documentation describing the extension. 
            * **externalDocs* (type: External Document Object or reference) - points to an external human-readable specification. May be specified inline, or may be a JSON reference to an ExternalDocumentation object defined in the same document (usually in a designated components location), or an external document. 
                * **description** - markdown description of the external documentation resource. 
                * **url** - URL resolving to the external documentation resource. 
            * **location** - URL that resolves to the authoritative definition of the OAS Extension descriptor. Supports the _Aggregation Catalog_ use case, providing the full specification along with a canonical location. 
            * **provider** (type: Provider Object or reference) - specifies some meta-information about the party responsible for defining the Extension. May be specified inline, or may be a JSON reference to a reusable component. 
                * **name** - name 
                * **(other)** - other properties, TBD 
            * **schema** (type: Schema Object or reference) - an OAS 3.0-compliant Schema Object describing the expected value of the extension property. May be specified inline, or may be a JSON reference thereto. 
            * **OAS2** (type: OAS2 Context Object) - Specifies usage in OAS2 context. 
                * **usage** - specifies whether usage is PROHIBITED, UNRESTRICTED, or RESTRICTED to set of object types, to be specified in the 'objectTypes' property. 
                * **objectTypes** (value: array of OAS2ObjectTypeName enum) - A list of OAS2 object types in which the extension property may be used. Valid only if 'usage' is RESTRICTED. 
            * **OAS3** (type: OAS3 Context Object) - Specifies usage in OAS3 context. 
                * **usage** - specifies whether usage is PROHIBITED, UNRESTRICTED, or RESTRICTED to set of object types, to be specified in the 'objectTypes' property. 
                * **objectTypes** (value: array of OAS3ObjectTypeName enum) - A list of OAS3 object types in which the extension property may be used. Valid only if 'usage' is RESTRICTED. 
    * **components** (type: Components Object) - collections of reusable components. 
        * **schemas** (type: Schemas Object) - catalog of reusable Schema Objects. 
            * **\[schemaName\]** (patterned field, type: Schema Object) - addressable object name. 
        * **providers** (type: Providers Object) - catalog of reusable Provider Objects. 
            * **\[providerName\]** (patterned field, type: Schema Object) - addressable object name. 
        * **externalDocs** (type: External Docs Object) - catalog of reusable External Documentation Objects. 
            * **\[documentName\]** (patterned field, type: Schema Object) - addressable object name.

## Examples

Here's an example 

```

