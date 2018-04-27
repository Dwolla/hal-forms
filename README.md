# Dwolla HAL Form Profile

* Version: 0.0.2
* Status: Draft

## Description
The [Dwolla API](https://docsv2.dwolla.com) uses the [HAL spec](https://tools.ietf.org/html/draft-kelly-json-hal-08) to represent resources. This profile acts as an extension in order to provide clients with the fields that are required to create or update entities. This allows for dynamically generated client-side forms that may change based on the state of the resource.

## Compliance

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119](https://tools.ietf.org/html/rfc2119).

## Media Type

This extension is to be used as a profile link ([RFC6906](https://tools.ietf.org/html/rfc6906)) in conjunction with a HAL style media type.

######Examples

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

## _links

Link attributes follow the same semantics as defined in the HAL specification [section 5](http://tools.ietf.org/html/draft-kelly-json-hal-06#section-5).

* **self**

  REQUIRED

  The resource that is being queried or modified.

## _forms

REQUIRED

An object containing one or many forms, denoted by key/value. Each key inside the `_forms` object MUST describe the form defined in its value. If only one form is returned, the keyword `default` SHOULD be used. It's RECOMMENDED to use keys that are verb-like in form (e.g., "create-customer", "search-customers") in order to provide context as to what the form is intended for.

### Form object

#### _links

REQUIRED

A set of links that describe the individual form.

* **target**

  REQUIRED

  A link that describes where to submit the form.

#### method

REQUIRED

The HTTP method to be used when submitting the resource generated from the form.

#### contentType

REQUIRED

The media type the API requires when submitting the resource generated from the form. This should be the value of the `Content-Type` header.

Clients MUST accept `application/x-www-form-urlencoded`, `multipart/form-data`, `application/json` and any media type ending in `+json`. Servers SHOULD NOT use media types outside this set.

Clients MUST follow JSON encoding rules for media types ending in `+json`.

### fields

REQUIRED

A collection of [field objects](#field-object).

### <a name="field-object"></a> Field object

A field object describes a value that MAY be submitted to the API.

### name

REQUIRED

The value of the property in the object being submitted to the API.

If a field is defined as follows:

```json
{
  "name": "firstName",
  "path": "/firstName",
  "type": "string"
}
```

Then the API is expecting an object that MAY contain `firstName`.

```json
{
  "firstName": "Jane Doe"
}
```

### path

OPTIONAL

A JSON Pointer ([RFC6901](http://tools.ietf.org/html/rfc6901)) to a field in a resource typically being created as a result of a submission to the API. The `path` property MAY be omitted when the submission results in a query instead of a new resource.

### value

OPTIONAL

The current/persisted value of the field.

### type

REQUIRED

Provides a hint indicating the type of values of this field and what UI element(s) would be appropriate to present to the user. This list MAY expand over time. Clients SHOULD treat unrecognized field types as `string`.

Possible types at this time:

* `boolean`

  True or false value.
  
  #### JSON Encoding
  
  Built-in boolean data type.
  
  #### Form Encoding
  
  `true` and `false`
  
* `number`

  Arbitrary precision decimal numbers.

  #### JSON encoding

  Built-in number datatype.
  
  #### Form Encoding
  
  UTF-8 encoded decimal number.

* `string`
  
  Short, probably single line, series of characters.
  
  #### JSON Encoding

  Built-in string datatype.
  
  #### Form Encoding
  
  UTF-8 encoded characters.

* `date`

   Calendar date representing a specific year, month and day. Dates are independent of time zones.
   
   #### JSON Encoding
   
   String containing exactly the [ISO 8601 encoded calendar date](https://en.wikipedia.org/wiki/ISO_8601#Calendar_dates)
   
   #### Form Encoding
   
   [ISO 8601 encoded calendar date](https://en.wikipedia.org/wiki/ISO_8601#Calendar_dates)
    
* `time`

  Time of day. Times MAY specify a time zone.

  #### JSON Encoding
  
  String containing exactly the [ISO 8601 encoded time](https://en.wikipedia.org/wiki/ISO_8601#Times)
  
  #### Form Encoding
  
  [ISO 8601 encoded time](https://en.wikipedia.org/wiki/ISO_8601#Times)
  
* `datetime`

  Calendar date and time. Datetimes MAY specify a time zone.

  #### JSON encoding
  
  String containing exactly the [ISO 8601 encoded date time](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations)
  
  #### Form Encoding
  
  [ISO 8601 encoded date time](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations)
  
* sensitive

  `string` whose value should be obscured in the user interfaces and logs.
  
  #### JSON Encoding
  
  Built-in string datatype.
  
  #### Form Encoding
  
  UTF-8 encoded characters.

* hidden
  
  Field needed by the form's target but that is not user editable.
  
  #### JSON Encoding
  
  Use the field `value` verbatim.
  
  #### Form Encoding
  
  Use form encoding rule for the field `value`'s datatype.
  
* text
  
  Potentially long, multi-line, string. Analogous to textarea in HTML.
  
  #### JSON Encoding
  
  Built-in string datatype.
  
  #### Form Encoding
  
  UTF-8 encoded characters.

* `email`

  Email address. Clients SHOULD encode email addresses as [RFC 6068 mailto URIs](https://tools.ietf.org/html/rfc6068). Servers MUST accept [RFC 6068 mailto URIs](https://tools.ietf.org/html/rfc6068). Servers SHOULD make a best effort attempt to extract an email address even if the value is not a `mailto` URI.
  
  #### JSON encoding
  
  String containing exactly an [RFC 6068 mailto URI](https://tools.ietf.org/html/rfc6068)
  
  #### Form Encoding
  
  [RFC 6068 mailto URI](https://tools.ietf.org/html/rfc6068)

* `tel`
  
  Telephone numbers. Clients SHOULD encode telephone numbers as [RFC 3966 telephone URIs](https://tools.ietf.org/html/rfc3966). Servers MUST accept [RFC 3966 telephone URIs](https://tools.ietf.org/html/rfc3966). Servers SHOULD make a best effort attempt to extract a phone  number from value even if it is not a `tel` URI.
  
  #### JSON Encoding
  
  String containing exactly an [RFC 3966 telephone URI](https://tools.ietf.org/html/rfc3966)
  
  #### Form Encoding
  
  [RFC 3966 telephone URI](https://tools.ietf.org/html/rfc3966)
  
* `file`

  File picker. Value will be the contents of the selected file.
  
  Forms using this field type MUST use `multipart/form-data` as their `contentType`.
  
  #### JSON encoding
  
  Unsupported. This field type MUST NOT be used with JSON encoding.
  
  #### Form Encoding

  Attach contents of file as one part of the multipart document as define by [RFC 7578](https://tools.ietf.org/html/rfc7578)


### displayText

OPTIONAL

A string that describes the field. This MAY be used in place of a client's own display text.

### validations

OPTIONAL

An object with rules that specify how the field MAY be validated.

Example validations:

```json
"validations": {
	"required": true,
	"regex": "^\d{3}-?\d{2}-?\d{4}$"
}
```

### accepted

OPTIONAL

An object that indicates a set of values accepted by the API for the given field. The value submitted MUST be one of the `value` properties from the list of `values` or `groupedValues`. The `values` and `groupedValues` properties MUST NOT appear at the same time in an `accepted` object within a field.

#### multiple

OPTIONAL

A boolean value which indicates whether multiple values MAY be submitted for the field. Defaults to `false` if not provided, meaning multiple values MUST NOT be submitted. If provided and `true`, values MUST be submitted as a collection/array of values.

#### values

OPTIONAL

A collection of [value objects](#value-object).

#### groupedValues

OPTIONAL

A collections of [grouped value objects](#grouped-value-object).

### <a name="value-object"></a> Value object

#### value

REQUIRED

The value that MUST be included in the submission to the API.

#### key

REQUIRED

A key that describes the value, which MAY be used for client side translation/display.

#### displayText

OPTIONAL

A string that describes the value. This MAY be used in place of a client's own display text for the accepted value.

### <a name="grouped-value-object"></a> Grouped value object

#### key

REQUIRED

A key that describes the grouping, which MAY be used for client side translation/display.

#### displayText

OPTIONAL

A string that describes the group. This MAY be used in place of a client's own display text for the accepted value.

#### values

REQUIRED

A collection of [value objects](#value-object).
