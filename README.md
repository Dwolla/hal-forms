# Dwolla HAL Form Profile

* Version: 0.0.2
* Status: Draft

## Description
The [Dwolla API](https://docsv2.dwolla.com) uses the [HAL spec](https://tools.ietf.org/html/draft-kelly-json-hal-08) to represent resources. This profile acts as an extension in order to provide clients with the fields that are required to create or update entities. This allows for dynamically generated client-side forms that may change based on the state of the resource.

## Compliance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119](https://tools.ietf.org/html/rfc2119).

JSON format is described using [MSON](https://github.com/apiaryio/mson).

## Media Type

This extension is to be used as a profile link ([RFC6906](https://tools.ietf.org/html/rfc6906)) in conjunction with a HAL style media type.

Examples

* `application/hal+json; profile="https://github.com/dwolla/hal-forms"`
* `application/vnd.dwolla.v1.hal+json; profile="https://github.com/dwolla/hal-forms"`

## Example Response

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

## HAL document form extension (object)

This spec extends the HAL format by adding the reserved property `_forms`. This property MAY appear on any HAL document (including embedded HAL documents).

### Properties

 - `_forms` (object, optional)

   - `default` ([form](#form), optional)

     The default form for the context. MAY be used when only one form makes sense in the context.

   - *form id(custom string)* ([form](#form), optional)

     A form related to the context. A verb first clauses expressing a command (such as, `create-customer`, `search-customers`) is RECOMMENDED for *form id*.

## <a id="form"/> Form (object)

A form object is a recipe for how to make a complex API requests.

### Properties

 - `_links`
   - `target` ([link object][], required)

     The resource which accepts submission of this form.

 - `method` (enum, required)

   The HTTP method to use when submitting this form. Consumers MUST ignore case of values.

   Value MUST be one of the following:
   - `GET`
   - `PUT`
   - `POST`
   - `PATCH`

 - `contentType` (string, optional)

   The media type the API requires when submitting the resource generated from the form. This should be the value of the `Content-Type` header. The `contentType` property is REQUIRED when `method` is anything except `GET`.

   Clients MUST accept `application/x-www-form-urlencoded`, `multipart/form-data`, `application/json` and any media type ending in `+json`. Servers SHOULD NOT use media types outside this set.

   Clients MUST follow JSON encoding rules for media types ending in `+json`.

   The HTTP method to be used when submitting the resource generated from the form.

 - `fields` (array[[field](#field)], required)

   The fields in this form.

## <a name="field"></a> Field (object)

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

### Properties

 - `name` (string, required)

   Identifies the field. Consumers SHOULD use this to locate fields of interest.

 - `path` (string, optional)

   A JSON Pointer ([RFC6901](http://tools.ietf.org/html/rfc6901)) to a field in a resource typically being created as a result of a submission to the API. The `path` property is REQUIRED when `contentType` is JSON. The `path` property SHOULD be omitted when the form's `method` is `GET` or the `contentType` is `application/x-www-form-urlencoded` or `multipart/form-data`.

 - value (*, required)

   The current/persisted value of the field.

 - `type` (enum, required)

   Provides a hint indicating the type of values of this field and what UI element(s) would be appropriate to present to the user. This list MAY expand over time. Clients SHOULD treat unrecognized field types as `string`.

   Possible types at this time:
   - `boolean`
     True or false value.

     #### JSON Encoding

     Built-in boolean data type.

     #### Form Encoding

     `true` and `false`

   - `number`

     Arbitrary precision decimal numbers.

     #### JSON encoding

     Built-in number datatype.

     #### Form Encoding

     UTF-8 encoded decimal number.

   - `string`

     Short, probably single line, series of characters.

     #### JSON Encoding

     Built-in string datatype.

     #### Form Encoding

     UTF-8 encoded characters.

   - `date`

     Calendar date representing a specific year, month and day. Dates are independent of time zones.

     #### JSON Encoding

     String containing exactly the [ISO 8601 encoded calendar date](https://en.wikipedia.org/wiki/ISO_8601#Calendar_dates)

     #### Form Encoding

     [ISO 8601 encoded calendar date](https://en.wikipedia.org/wiki/ISO_8601#Calendar_dates)

   - `time`

     Time of day. Times MAY specify a time zone.

     #### JSON Encoding

     String containing exactly the [ISO 8601 encoded time](https://en.wikipedia.org/wiki/ISO_8601#Times)

     #### Form Encoding

     [ISO 8601 encoded time](https://en.wikipedia.org/wiki/ISO_8601#Times)

   - `datetime`

     Calendar date and time. Datetimes MAY specify a time zone.

     #### JSON encoding
     
     String containing exactly the [ISO 8601 encoded date time](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations)
     
     #### Form Encoding
     
     [ISO 8601 encoded date time](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations)
     
   - sensitive

     `string` whose value should be obscured in the user interfaces and logs.
     
     #### JSON Encoding
     
     Built-in string datatype.
     
     #### Form Encoding
     
     UTF-8 encoded characters.

   - hidden
     
     Field needed by the form's target but that is not user editable.
     
     #### JSON Encoding
     
     Use the field `value` verbatim.
     
     #### Form Encoding
     
     Use form encoding rule for the field `value`'s datatype.
     
   - text
     
     Potentially long, multi-line, string. Analogous to textarea in HTML.
     
     #### JSON Encoding
     
     Built-in string datatype.
     
     #### Form Encoding
     
     UTF-8 encoded characters.

   - `email`

     Email address. Clients SHOULD encode email addresses as [RFC 6068 mailto URIs](https://tools.ietf.org/html/rfc6068). Servers MUST accept [RFC 6068 mailto URIs](https://tools.ietf.org/html/rfc6068). Servers SHOULD make a best effort attempt to extract an email address even if the value is not a `mailto` URI.
     
     #### JSON encoding
     
     String containing exactly an [RFC 6068 mailto URI](https://tools.ietf.org/html/rfc6068)
     
     #### Form Encoding
     
     [RFC 6068 mailto URI](https://tools.ietf.org/html/rfc6068)

   - `tel`
     
     Telephone numbers. Clients SHOULD encode telephone numbers as [RFC 3966 telephone URIs](https://tools.ietf.org/html/rfc3966). Servers MUST accept [RFC 3966 telephone URIs](https://tools.ietf.org/html/rfc3966). Servers SHOULD make a best effort attempt to extract a phone  number from value even if it is not a `tel` URI.
     
     #### JSON Encoding
     
     String containing exactly an [RFC 3966 telephone URI](https://tools.ietf.org/html/rfc3966)
     
     #### Form Encoding
     
     [RFC 3966 telephone URI](https://tools.ietf.org/html/rfc3966)
     
   - `file`

     File picker. Value will be the contents of the selected file.
     
     Forms using this field type MUST use `multipart/form-data` as their `contentType`.
     
     #### JSON encoding
     
     Unsupported. This field type MUST NOT be used with JSON encoding.
     
     #### Form Encoding

     Attach contents of file as one part of the multipart document as define by [RFC 7578](https://tools.ietf.org/html/rfc7578)

 - `displayText` (string, optional)

   A human readable string that describes the field. This MAY be used in place of a client's own display text. Clients SHOULD use `name` if this is missing.

 - `validations` ([validation](#validation), optional)

   An object with rules specifying how value of the field MAY be validated.

 - `accepted` (object, optional)

   An object that indicates the exhaustive set of values accepted by the API for the field.

   - One Of
     - `values` (array[[value](#value)], required)

       Simple list of values acceptable values.

     - `groupedValues` (array[[value group](#value-group)], required)

       List of groups of values that are acceptable.

 - `multiple` (boolean, optional)

   A boolean value which indicates whether multiple values MAY be submitted for the field. Defaults to `false` if not provided, meaning multiple values MUST NOT be submitted. If provided and `true`, values MUST be submitted as a collection/array of values.

## <a id="validation"/> Validation (object)

A description of syntactic restrictions on the containing [field](#field). This information MAY be used by the client to provide feedback to users without requiring a round trip to the server. API providers MUST validate submitted forms for validity regardless of the validation information embedded in the form.

Example validation object
```json
"validations": {
	"required": true,
	"regex": "^\d{3}-?\d{2}-?\d{4}$"
}
```

### Properties

 - `required` (boolean, optional)

   True indicates that this field is required in submitted forms. Default value is false.

 - `regex` (string, optional)

   A [Perl compatible regular expression](https://en.wikipedia.org/wiki/Perl_Compatible_Regular_Expressions) describing allowed syntax of this field's values. API consumers MUST ignore this property unless the field type is `string` or `text`


## <a id="value-group"/> Value group (object)

A group of valid values for the containing [field](#field).

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

### Properties
 - `key` (string, required)

   An identifier for the value. Consumers MAY use this for identification and display.

 - `displayText` (string, optional)

   A human readable description of this value. If present, consumers SHOULD use this when displaying the value to users.

 - `values` (array[[value](#value)], required)

   List of sub-values, one of which must be used in the form submission if this value is selected.


## <a id="value"/> Value (object)

A single valid value for the containing [field](#field).

```json
{
  "value": "breweries",
  "key": "BREWERIES",
  "displayText": "Breweries"
},
```

### Properties
 - `key` (string, required)

   An identifier for the value. Consumers MAY use this for identification and display.

 - `displayText` (string, optional)

   A human readable description of this value. If present, consumers SHOULD use this when displaying the value to users.

 - `value` (*, required)

   The value that MUST be used in the form submission if this value is selected.






[link object]: https://tools.ietf.org/html/draft-kelly-json-hal-08#page-4