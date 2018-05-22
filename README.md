# Dwolla HAL Form Profile

* Version: 0.0.2
* Status: Draft

## Description
This specification defines an extension to the [HAL json format](https://tools.ietf.org/html/draft-kelly-json-hal-08) to improve the evolvability of HAL based APIs.

## Compliance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119](https://tools.ietf.org/html/rfc2119).

JSON format is described using [MSON](https://github.com/apiaryio/mson).

## Table of Contents

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [Dwolla HAL Form Profile](#dwolla-hal-form-profile)
    - [Description](#description)
    - [Compliance](#compliance)
    - [Table of Contents](#table-of-contents)
    - [Introduction](#introduction)
    - [Media Type](#media-type)
    - [Document structure](#document-structure)
        - [HAL document form extension (object)](#hal-document-form-extension-object)
            - [Properties](#properties)
        - [Form (object)](#form-object)
            - [Properties](#properties-1)
        - [Field (object)](#field-object)
            - [Properties](#properties-2)
        - [Validation (object)](#validation-object)
            - [Properties](#properties-3)
        - [Value group (object)](#value-group-object)
            - [Properties](#properties-4)
        - [Value (object)](#value-object)
            - [Properties](#properties-5)
    - [Processing](#processing)
        - [Target URL resolution](#target-url-resolution)
        - [Body construction](#body-construction)
            - [Form transcoding](#form-transcoding)
                - [Value transcoding](#value-transcoding)
            - [JSON transcoding](#json-transcoding)
                - [Value transcoding](#value-transcoding-1)

<!-- markdown-toc end -->

## Introduction

Evolvability is important for any API with more than one client. The HAL specification requires API consumers have deep knowledge of the domain and implementation details of the API provider in situations that require user input or resulting unsafe requests. This deep knowledge inherently couples API consumers to the provider(s) reducing the evolvability of the overall system.

Forms are well tested way to reduce the coupling between API consumers and providers. This form system takes a highly pragmatic approach similar to the approach of HAL to APIs.

## Media Type

This extension is to be used as a profile link ([RFC6906](https://tools.ietf.org/html/rfc6906)) in conjunction with a HAL style media type.

Examples

* `application/hal+json; profile="https://github.com/dwolla/hal-forms"`
* `application/vnd.dwolla.v1.hal+json; profile="https://github.com/dwolla/hal-forms"`

## Document structure

### HAL document form extension (object)

This spec extends the HAL format by adding the reserved property `_forms`. This property MAY appear on any HAL document (including embedded HAL documents).

Example HAL document with a form
```json
{
  "_links": {
    "self": {
      "href": "http://api.example.com/customers"
    }
  },
  "_forms": {
    "default": {
      "_links": {
        "target": {
          "href": "http://api.example.com/customers"
        }
      },
      "method": "POST",
      "contentType": "application/hal+json",
      "fields": [
        {
          "name": "name",
          "path": "/name",
          "type": "string",
          "value": "Dwolla",
          "displayText": "Name",
          "validations": {
            "required": true
          }
        },
        {
          "name": "email",
          "path": "/email",
          "type": "email",
          "displayText": "Email",
          "validations": {
            "required": true
          }
        },
        {
          "name": "password",
          "path": "/password",
          "type": "sensitive",
          "displayText": "Password",
          "validations": {
            "required": true
          }
        },
        {
          "name": "businessType",
          "path": "/businessType",
          "type": "string",
          "displayText": "Business Type",
          "validations": {
            "required": true
          },
          "accepted": {
            "values": [
              {
                "value": "corporation",
                "key": "CORPORATION",
                "displayText": "Corporation"
              },
              {
                "value": "llc",
                "key": "LLC",
                "displayText": "LLC"
              },
              {
                "value": "partnership",
                "key": "PARTNERSHIP",
                "displayText": "Partnership"
              },
              {
                "value": "soleproprietorship",
                "key": "SOLEPROPRIETORSHIP",
                "displayText": "Sole Proprietorship"
              }
            ]
          }
        },
        {
          "name": "businessClassification",
          "path": "/businessClassification",
          "type": "string",
          "displayText": "Business Classification",
          "validations": {
            "required": true
          },
          "accepted": {
            "groupedValues": [
              {
                "key": "FOOD_RETAIL_AND_SERVICE",
                "displayText": "Food retail and service",
                "values": [
                  {
                    "value": "breweries",
                    "key": "BREWERIES",
                    "displayText": "Breweries"
                  },
                  {
                    "value": "distilleries",
                    "key": "DISTILLERIES",
                    "displayText": "Distilleries"
                  }
                ]
              },
              {             
                "key": "MANUFACTURING",
                "displayText": "Manufacturing",
                "values": [
                  {
                    "value": "computers",
                    "key": "COMPUTER_AND_ELECTRONIC_PRODUCT_MANUFACTURING",
                    "displayText": "Computer and electronic product manufacturing"
                  },
                  {
                    "value": "furniture",
                    "key": "FURNITURE_AND_RELATED_PRODUCT_MANUFACTURING",
                    "displayText": "Furniture and related product manufacturing"
                  }
                ]
              }
            ]
          }
        }
      ]
    }
  }
}
```


#### Properties

 - `_forms` (object, optional)

   - `default` ([form](#form), optional)

     The default form for the context. API providers SHOULD use this to indicate the best form for consumers with limited domain knowledge.

   - *form id(custom string)* ([form](#form-object), optional)

     A form related to the context. A verb first clauses expressing a command (such as, `create-customer`, `search-customers`) is RECOMMENDED for *form id*.

### Form (object)

A form object is a recipe for making complex API requests.

#### Properties

 - `_links`
   - `target` ([link object][], required)

     The resource which accepts submission of this form.  The target of a form MAY be a [templated link](https://tools.ietf.org/html/draft-kelly-json-hal-08#section-5.2

 - `method` (enum, required)

   The HTTP method to use when submitting this form. Consumers MUST ignore case of values.

   Value MUST be one of the following:
   - `GET`
   - `DELETE`
   - `PATCH`
   - `POST`
   - `PUT`

 - `contentType` (string, optional)

   The media type of submissions required by the target resource. This should be the value of the `Content-Type` header. The `contentType` property is REQUIRED when `method` is  `PATCH`, `POST` or `PUT`.

   Clients MUST accept `application/x-www-form-urlencoded`, `multipart/form-data`, `application/json` and any media type ending in `+json`. Servers SHOULD NOT use media types outside this set.

   Clients MUST follow JSON encoding rules for media types ending in `+json`.

 - `fields` (array[[field](#field-object)], required)

   The fields in this form.

### Field (object)

A field object describes a value that can be submitted as part of this form.

Example field object
```json
{
  "name": "name",
  "path": "/name",
  "type": "string",
  "value": "Dwolla",
  "displayText": "Name",
  "validations": {
    "required": true
  }
}
```

#### Properties

 - `name` (string, required)

   Identifies the field. Consumers MAY use this to locate fields of interest.

 - `path` (string, optional)

   A JSON Pointer ([RFC6901](http://tools.ietf.org/html/rfc6901)) to a field in a resource typically being created as a result of a submission to the API. The `path` property is REQUIRED when `contentType` is JSON. The `path` property SHOULD be omitted when the form's `method` is `GET` or `DELETE` or the `contentType` is `application/x-www-form-urlencoded` or `multipart/form-data`.

 - value (*, optional)

   The current/persisted value of the field.

 - `type` (enum, required)

   Provides a hint indicating the type of values of this field and what UI element(s) would be appropriate to present to the user. This list MAY expand over time. Clients SHOULD treat unrecognized field types as `string`.

   Possible types at this time:
   - `boolean`
 
     True or false value.

   - `number`

     Arbitrary precision decimal numbers.

   - `string`

     Short, probably single line, series of characters. Consumers should present the user with a single line text entry box.

   - `date`

     Calendar date representing a specific year, month and day. Dates are independent of time zones.

   - `time`

     Time of day. Times MAY specify a time zone.

   - `datetime`

     Calendar date and time. Datetimes MAY specify a time zone.

   - sensitive

     `string` whose value should be obscured in the user interfaces and logs.

   - `hidden`

     Field needed by the form's target but that is not user editable.

   - `text`

     Potentially long, multi-line, string. Analogous to textarea in HTML. Consumers should present the user with a multi-line text entry area.

   - `email`

     Email address.

   - `tel`

     Telephone numbers. API consumers SHOULD support international phone numbers.

   - `file`

     File picker. Value will be the contents of the selected file.

     Forms using this field type MUST use `multipart/form-data` as their `contentType`.

 - `displayText` (string, optional)

   A human readable string that describes the field. This SHOULD be used in place of a client's own display text. Clients SHOULD use `name` if this is missing.

 - `validations` ([validation](#validation-object), optional)

   An object with rules specifying how values of the field MAY be validated.

 - `accepted` (object, optional)

   An object that indicates the exhaustive set of values accepted by the API for the field.

   - One Of
     - `values` (array[[value](#value-object)], required)

       Simple list of acceptable values.

     - `groupedValues` (array[[value group](#value-group-object)], required)

       List of groups of values that are acceptable.

 - `multiple` (boolean, optional)

   A boolean value which indicates whether multiple values MAY be submitted for the field. Defaults to `false` if not provided, meaning multiple values MUST NOT be submitted. If provided and `true`, values MUST be submitted as a collection/array of values.

### Validation (object)

A description of syntactic restrictions on the containing [field](#field-object). This information MAY be used by the client to provide feedback to users without requiring a round trip to the server. API providers MUST validate submitted forms for validity regardless of the validation information embedded in the form.

Example validation object
```json
"validations": {
	"required": true,
	"regex": "^\d{3}-?\d{2}-?\d{4}$"
}
```

#### Properties

 - `required` (boolean, optional)

   True indicates that forms submitted without a value for this field will be rejected by the API provider. API consumers SHOULD treat this as false if it is absent.

 - `regex` (string, optional)

   A [Perl compatible regular expression](https://en.wikipedia.org/wiki/Perl_Compatible_Regular_Expressions) describing allowed syntax of this field's values. API providers SHOULD omit this property unless the field type is `string` or `text`. API consumers MUST ignore this property unless the field type is `string` or `text`


### Value group (object)

A group of valid values for the containing [field](#field-object).

Example of value group
```json
{
  "key": "MANUFACTURING",
  "displayText": "Manufacturing",
  "values": [
    {
      "value": "computers",
      "key": "COMPUTER_AND_ELECTRONIC_PRODUCT_MANUFACTURING",
      "displayText": "Computer and electronic product manufacturing"
    },
    {
      "value": "furniture",
      "key": "FURNITURE_AND_RELATED_PRODUCT_MANUFACTURING",
      "displayText": "Furniture and related product manufacturing"
    }
  ]
}
```

#### Properties
 - `key` (string, required)

   An identifier for the group of values. Consumers MAY use this for identification and display.

 - `displayText` (string, optional)

   A human readable description of this value. If present, consumers SHOULD use this when displaying the value to users.

 - `values` (array[[value](#value-object)], required)

   List of values, one of which must be used in the form submission if this value group is selected.


### Value (object)

A single valid value for the containing [field](#field).
```json
{
  "value": "breweries",
  "key": "BREWERIES",
  "displayText": "Breweries"
},
```

#### Properties
 - `key` (string, required)

   An identifier for the value. Consumers MAY use this for identification and display.

 - `displayText` (string, optional)

   A human readable description of this value. If present, consumers SHOULD use this when displaying the value to users.

 - `value` (*, required)

   The value that MUST be used in the form submission if this value is selected.

## Processing

To submit a form API consumers will
 1. collect information for each field from the user
 1. [resolve the target URL](#target-url-resolution)
 1. [construct submission body](#body-construction)
 1. make submission request

The submission requests MUST
 - be to the resolved target URL
 - use the `method` specified by the form
 - have a body if the method allows it
 - have a `Content-Type` header field whose value is the `contentType` of the form if there is a body

### Target URL resolution

The target of a form MAY be a [templated link](https://tools.ietf.org/html/draft-kelly-json-hal-08#section-5.2). If the target link is not templated it's `href` MUST be used verbatim.

When target link is a template it MUST be expanded using the form fields in order to resolve it into a actual URL. Form fields must be converted in to a set of variable definitions whose values are the field values encoded using the [form value transcoding rules](#value-transcoding). The templated is then expanded using that set of variable definitions using the process define by [RFC 6570](https://tools.ietf.org/html/rfc6570).

For example, this form
```json
{ "_links": {
    "target" : {
      "href": "http://example.com/customers{?cust_id,name}",
      "templated": true
    }
  },
  "contentType": "application/x-www-form-urlencoded",
  "method": "GET",
  "fields": [
    { "name": "cust_id",
      "type": "string" },
    { "name": "name",
      "type": "string" }
  ]
}
```
might resolve to any of the following depending on the user input
 - `http://example.com/customers?cust_id=42`
 - `http://example.com/customers?name=frolic`
 - `http://example.com/customers?cust_id=42&name=frolic`

A field MAY be used in both URL expansion and body construction.

API producers MUST NOT include fields on forms whose method is GET or DELETE and whose target link is non-templated. Client SHOULD ignore fields on forms whose method is GET or DELETE and whose target link is non-templated.

### Body construction

If the form's method allows a body (`PUT`, `POST` or `PATCH`) then the form is used to construct a body document for the form submission. The exact algorithm for body construction varies based on the content type of the form. Forms with `application/x-www-form-urlencoded` or `multipart/form-data` content types use [form transcoding](#form-transcoding). Forms with `application/json` or `+json` content types use [JSON transcoding](#json-transcoding).

#### Form transcoding

Form encoding applies to forms whose `contentType` is `application/x-www-form-urlencoded` or `multipart/form-data`. For form encoding each field of the form is as a name-value pair. Values are converted into a string base on type specific rules specified below. The resulting name-value pairs are then encoding following common encoding rules for `application/x-www-form-urlencoded` or [RFC7578](https://tools.ietf.org/html/rfc7578) for `multipart/form-data`.

For example, the following `x-www-form-urlencoded` form
```json
{ "_links": { "target" : { "href": "http://example.com" } },
  "contentType": "application/x-www-form-urlencoded",
  "method": "POST",
  "fields": [
    { "name": "title",
      "type": "string" },
    { "name": "recommended",
      "type": "boolean" }
  ]
}
```

might encode into this body
```
title=User+Provided+Title&recommended=true
```

With similar user input the following `form-data` form
```json
{ "_links": { "target" : { "href": "http://example.com" } },
  "contentType": "multipart/form-data",
  "method": "POST",
  "fields": [
    { "name": "title",
      "type": "string" },
    { "name": "recommended",
      "type": "boolean" }
  ]
}
```

encodes into this body
```
--AaB03x
content-disposition: form-data; name="title"

User Provided Title
--AaB03x
content-disposition: form-data; name="recommended"

true
--AaB03x
```

##### Value transcoding
 - `boolean`

   literal `true` and `false` UTF-8 strings

 - `number`

    Decimal number encoded in UTF-8.

 - `string`

   UTF-8 encoded characters.

 - `date`

    Clients SHOULD encode dates as [ISO 8601 encoded calendar date](https://en.wikipedia.org/wiki/ISO_8601#Calendar_dates). Servers MUST accept [ISO 8601 encoded calendar date](https://en.wikipedia.org/wiki/ISO_8601#Calendar_dates). Servers MAY make a best effort attempt to extract a date even if the value is not a valid ISO 8601 date.

 - `time`

    Clients SHOULD encode times as [ISO 8601 encoded time](https://en.wikipedia.org/wiki/ISO_8601#Times). Servers MUST accept [ISO 8601 encoded time](https://en.wikipedia.org/wiki/ISO_8601#Times). Servers MAY make a best effort attempt to extract a time even if the value is not a valid ISO 8601 time.

 - `datetime`

    Clients SHOULD encode date times as [ISO 8601 encoded date time](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations). Servers MUST accept [ISO 8601 encoded date time](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations). Servers MAY make a best effort attempt to extract a date and time even if the value is not a valid ISO 8601 date time.

 - sensitive

   UTF-8 encoded characters.

 - hidden

   Use value encoding rule for the field `value`'s datatype.

 - text

   UTF-8 encoded characters.

 - `email`

   Clients SHOULD encode email addresses as [RFC 6068 mailto URI](https://tools.ietf.org/html/rfc6068). Servers MUST accept [RFC 6068 mailto URIs](https://tools.ietf.org/html/rfc6068). Servers SHOULD make a best effort attempt to extract an email address even if the value is not a `mailto` URI.

 - `tel`

   Clients SHOULD encode telephone numbers as [RFC 3966 telephone URIs](https://tools.ietf.org/html/rfc3966). Servers MUST accept [RFC 3966 telephone URIs](https://tools.ietf.org/html/rfc3966). Servers SHOULD make a best effort attempt to extract a phone  number from value even if it is not a `tel` URI.

 - `file`

   File attached as one part of the multipart document as define by [RFC 7578](https://tools.ietf.org/html/rfc7578)
   Forms using this field type MUST use `multipart/form-data` as their `contentType`.


#### JSON transcoding

JSON transcoding applies to forms whose `contentType` is `application/json` or any media type ending in `+json`. Field values are converted into a native JSON data type using the value trancoding rules below. Those encoded values are then added to a JSON document at the location indicated by the `path` of the [field](#field-object). The body is complete once all fields have been inserted.

For example, the following JSON form
```json
{ "_links": { "target" : { "href": "http://example.com" } },
  "contentType": "application/x-www-form-urlencoded",
  "method": "POST",
  "fields": [
    { "name": "title",
      "type": "string",
      "path": "/title" },
    { "name": "recommended",
      "type": "boolean",
      "path": "/superfluous/nesting/recommended" }
  ]
}
```

would encode into this body
```
{ "title": "User Provided Title",
  "superfluous": {
    "nesting": {
      "recommended": true
    }
  }
}
```

##### Value transcoding

 - `boolean`
   Built-in boolean data type.

 - `number`

   Built-in number datatype.

 - `string`

   Built-in string datatype.

 - `date`

    Clients SHOULD encode dates as [ISO 8601 encoded calendar date](https://en.wikipedia.org/wiki/ISO_8601#Calendar_dates). Servers MUST accept [ISO 8601 encoded calendar date](https://en.wikipedia.org/wiki/ISO_8601#Calendar_dates). Servers MAY make a best effort attempt to extract a date even if the value is not a valid ISO 8601 date.

 - `time`

    Clients SHOULD encode times as [ISO 8601 encoded time](https://en.wikipedia.org/wiki/ISO_8601#Times). Servers MUST accept [ISO 8601 encoded time](https://en.wikipedia.org/wiki/ISO_8601#Times). Servers MAY make a best effort attempt to extract a time even if the value is not a valid ISO 8601 time.

 - `datetime`

    Clients SHOULD encode date times as [ISO 8601 encoded date time](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations). Servers MUST accept [ISO 8601 encoded date time](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations). Servers MAY make a best effort attempt to extract a date and time even if the value is not a valid ISO 8601 date time.

 - sensitive

   Built-in string datatype.

 - hidden

   Use the field `value` verbatim.

 - text

   Built-in string datatype.

 - `email`

   Clients SHOULD encode email addresses as [RFC 6068 mailto URIs](https://tools.ietf.org/html/rfc6068). Servers MUST accept [RFC 6068 mailto URIs](https://tools.ietf.org/html/rfc6068). Servers SHOULD make a best effort attempt to extract an email address even if the value is not a `mailto` URI.

 - `tel`

   Clients SHOULD encode telephone numbers as [RFC 3966 telephone URIs](https://tools.ietf.org/html/rfc3966). Servers MUST accept [RFC 3966 telephone URIs](https://tools.ietf.org/html/rfc3966). Servers SHOULD make a best effort attempt to extract a phone  number from value even if it is not a `tel` URI.

 - `file`

   Unsupported. This field type MUST NOT be used with a JSON `contentType`.



[link object]: https://tools.ietf.org/html/draft-kelly-json-hal-08#page-4

