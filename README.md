# SEMOASA - Specification Extension Metadata for OAS Applications (DRAFT)
## Background
The OpenAPI Specification, formerly known as the Swagger Specification, is a standard, machine-readable format for REST-style API definitions and documentation. Swagger 2.0 introduced _vendor extensions_, which allow certain objects to have properties named with an `x-` prefix, with arbitrary or independently specified property values. Vendor extensions were later renamed to [specification extensions](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#specificationExtensions) in OpenAPI 2.0 and 3.0.

## Motivation

While specification extensions are [widely used](https://github.com/Mermade/openapi-specification-extensions/blob/master/extensions/combined.tsv), there is no standardized way to define their syntax, expected values, and allowed usage context. This makes it much more difficult for OpenAPI tools to support specification extensions with content assist, validation, and other features that those tools typically provide for standard OpenAPI language constructs. 

For example, an OpenAPI editor will usually have design-time knowledge of the OpenAPI specification, and _may_ make use of a JSON Schema that encodes some of this knowledge. Based on this knowledge, it can provide context-sensitive editing assistance and validation. But if the user adds extended properties with the `x-` prefix, the trail goes cold. Unless the editor has been designed with explicit prior knowledge of that specification extension, it can't tell the user what the extended property means, what kinds of values are expected, and whether that extension is even meaningful in this context. 

SEMOASA is a machine-readable format for specification extensions that provides a base level of structural and descriptive information, along with optional links to external resources for human-readable documentation. Like OpenAPI itself, specification extensions may have certain constraints and semantics that aren't captured in a schema or metadata descriptor; but the information carried in SEMOASA is sufficient to allow OpenAPI tools to provide a user experience much closer to what they provide for standard OpenAPI syntax.

## Use Cases

### Roles

SEMOSA use cases involve interactions among systems or system implementors in different roles:
* The _extension provider_ (or simply _provider_) defines one or more specification extension properties, and usually provides software that uses these properties. The provider may be a vendor, an open source software project, or a standards body (formal or informal). In this scenario, the provider may specify a URL as a _canonical source location_ to access the SEMOASA document.
* The _publisher_ exposes a SEMOASA document describing a single specification extension property or a _catalog_ of extension properties. The publisher may be an extension provider, publishing its own extension properties in a SEMOASA document. Or it may be a _registry_, publishing a catalog of third-party extensions. 
* The _consumer_ is a software program that reads a SEMOASA document, and makes uses the specification metadata. Consumers may be OpenAPI editors, validators, code generators, automated build tools, middleware integration modules, or other processors that work with OpenAPI documents.

### **Catalog Patterns** 
Publishers and consumers may want to exchange catalogs in different forms:
* *Directory Catalog* - A publisher may provide a list of known OpenAPI specification extensions, referring back to the canonical source locations for the actual extension metadata.  A consumer of a Directory Catalog would need traverse references to those locations to retrieve the metadata. 
* *Aggregation Catalog* - A publisher may provide a collection of known OpenAPI specification extensions, including the full extension metadata. This allows a consumer to ingest a full set of metadata without requiring additional round trips. In this case, it should still be possible to include the canonical location, so the consumer can refresh these definitions as needed.  

### **Tool Functions**
SEMOASA is designed with a few representative use cases in mind
* *Content Assist* - Providing in-place code assist, or "Intellisense", including proposals for specification extension properties intended for use in the current context. 
* *Tool Tips* - Extension property descriptions and/or formatted documentation, visible when hovering over an actual or proposed usage of the extension property. 
* *Validation* - Checking usage of extension properties against the provided extension descriptor, flagging errors or warnings where an extended property appears in an unexpected context, or has a value that does not conform to the specified schema. 

## Design Goals
SEMOASA aims to meet the following goals, to the extent practical:
* **Consistency with OpenAPI**, so developers familiar with current OpenAPI concepts, terminology and style will be familiar with corresponding elements of SEMOASA.
* **Common Technology Stack with OpenAPI**, so that essential functions like YAML editing, schema validation, and JSON reference resolution can be performed by the same components used with OpenAPI tool implementations. 
* **Support for OpenAPI 2.0 and later**, to ensure relevancy with current and future usage.
* **Provider-Neutrality**, not being significantly influenced by a particular set of tools or OpenAPI specification extension providers.
* **Extension Identity**, allowing consumers to disambiguate duplicate names. Consumers may import extension metadata from multiple publishers, with overlapping catalogs. Since extension providers are not required to reserve unique property names in a central registry, name clashes can occur. To meet the identity requirement, SEMOASA introduces _namespace_ to define a set of extension properties under coordinated management.

## Structure

Once the SEMOASA format gets some traction and achieves a level of stability, we'll write a formal specification. For now, this outline format summarizes the structure of a SEMOASA document:

* **Extensions Object** - an object defining one or more extensions, organized by namespace. 
    * **openapiExtensionFormat** - a version string indicating the format used to describe the extensions. Allows tools to support multiple versions of specification extensions. Each revision of the specification will prescribe a specific version to use. Should adhere to semantic versioning, allowing tools to support a given major or minor version. 
    * **\[namespace\]** (patterned field, type: Namespace Object or reference) - a namespace encompassing a set of managed Specification Extensions. Within a given namespace, management of Specification Extensions is expected to be quasi-coordinated, at least to the extent of ensuring unique extension names. As a convention, namespaces should generally use reverse-DNS notation, with additional segments as needed, e.g. `com.adobe.experience.public`. (Pattern for this set of properties can enforce that convention, and thus exclude statically defined properties like openapiExtensionFormat.) May contain a set of Extension Descriptors, or a JSON Reference to a Namespace Object, to support the _Directory Catalog_ use case.
        * **\[extensionPropertyName\]** (patterned field, type: Extension Object or reference)- name of the specification extension property. must start with "x-" as per the OAS specification. We may want to support wildcards or template parameters in the name. May contain an Extension Object or a JSON Reference thereto, supporting the _Directory Catalog_ use case (one level deeper than namespace). 
            * **summary** - a plain-text summary of the purpose of the extension. 
            * **description** - markdown documentation describing the extension. 
            * **externalDocs** (type: External Document Object or reference) - points to an external human-readable specification. May be specified inline, or may be a JSON reference to an ExternalDocumentation object defined in the same document (usually in a designated components location), or an external document. 
                * **description** - markdown description of the external documentation resource. 
                * **url** - URL resolving to the external documentation resource. 
            * **location** - URL that resolves to the authoritative definition of the OAS Extension descriptor. Supports the _Aggregation Catalog_ use case, providing the full specification along with a canonical location. 
            * **provider** (type: Provider Object or reference) - specifies some meta-information about the party responsible for defining the Extension. May be specified inline, or may be a JSON reference to a reusable component. 
                * **name** - name 
                * **(other)** - other properties, TBD 
            * **schema** (type: Schema Object or reference) - an OAS 3.0-compliant Schema Object describing the expected value of the extension property. May be specified inline, or may be a JSON reference thereto. 
            * **oas2** (type: OAS2 Context Object) - Specifies usage in OAS2 context. 
                * **usage** - specifies allowed usage as one of the following:
                    - `prohibited` - may not be used in oas2.
                    - `unrestricted` - may be used in any oas2 object that allows specification extension properties. 
                    - `restricted` - may be used only in objects of the types types be specified in the 'objectTypes' property.
                * **objectTypes** (value: array of OAS2ObjectTypeName enum) - A list of OAS2 object types in which the extension property may be used. Valid only if 'usage' is RESTRICTED. 
            * **oas3** (type: OAS3 Context Object) - Specifies usage in OAS3 context. 
                * **usage** - specifies allowed usage as one of the following:
                    - `prohibited` - may not be used in oas3.
                    - `unrestricted` - may be used in any oas3 object that allows specification extension properties. 
                    - `restricted` - may be used only in objects of the types types be specified in the 'objectTypes' property.
                * **objectTypes** (value: array of OAS3ObjectTypeName enum) - A list of OAS3 object types in which the extension property may be used. Valid only if 'usage' is RESTRICTED. 
    * **components** (type: Components Object) - collections of reusable components. 
        * **schemas** (type: Schemas Object) - catalog of reusable Schema Objects. 
            * **\[schemaName\]** (patterned field, type: Schema Object) - addressable object name. 
        * **providers** (type: Providers Object) - catalog of reusable Provider Objects. 
            * **\[providerName\]** (patterned field, type: Schema Object) - addressable object name. 
        * **externalDocs** (type: External Docs Object) - catalog of reusable External Documentation Objects. 
            * **\[documentName\]** (patterned field, type: Schema Object) - addressable object name.

## Examples

Here's a starting example:

```yaml
openapiExtensionFormat: 0.1.0
com.amazon.aws:
  x-amazon-apigateway-integration:
    summary: Specifies the integration of the method with the backend.
    description: |
      Specifies details of the backend integration used for this method. 
      This extension is an extended property of the Swagger Operation object.
      The result is an API Gateway integration object.
    externalDocs:
      description: AWS documentation page in the  Amazon API Gateway Developer Guide 
      url: http://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-swagger-extensions-integration.html
    provider:
      name: Amazon Web Services
      url: https://aws.amazon.com/
    schema: 
      type: object
      properties:
        cacheKeyParameters:
          type: array
          items: string
          description: A list of request parameters whose values are to be cached.
      cacheNamespace:
        type: string
        description: An API-specific tag group of related cached parameters.
      credentials:
        type: string  
        description: |
          For AWS IAM role-based credentials, specify the ARN of an appropriate IAM role. 
          If unspecified, credentials will default to resource-based permissions that must
          be added manually to allow the API to access the resource. For more information,
          see Granting Permissions Using a Resource Policy. 
      contentHandling: 
        type: string
        description: |
          Request payload encoding conversion types. Valid values are 
          1) CONVERT_TO_TEXT, for converting a binary payload into a Base64-encoded string 
          or converting a text payload into a utf-8-encoded string or passing through the 
          text payload natively without modification, and 
          2) CONVERT_TO_BINARY, for converting a text payload into Base64-decoded blob or 
          passing through a binary payload natively without modification.
      httpMethod:
        type:  string
        description: |
          The HTTP method used in the integration request. For Lambda function invocations, 
          the value must be POST.
      #... (other properties)
    oas2:
      usage: restricted
      objectTypes:
      - OperationObject
    oas3:
      usage: restricted
      objectTypes:
      - OperationObject
```

## Implementations 

Coming soon!

## Contributing to SEMOASA

Please comment! This is an early DRAFT specification. We need input from OAS tool providers, extension providers, the OpenAPI TDC, and other interested parties.

Please review the [open issues](https://github.com/RepreZen/SEMOASA/issues), feel free to comment on these, and open new issues.
