## How to create docs for your model/api

There are two ways to do so:
- comment based description;
- annotation based;

### Comment based

In comment based way you should add comments above the property you want to add a description. Valid properties are: message, rpc, message and field.
Comments are divided into two paragraphs: first paragraph will be a title, second one - summary. It is less verbose way, hence easier to read and understand.
This one is preferred for dealing with plain cases to describe the properties in API.

#### An example:
In `.proto` file:
```
  ...
  //content
  //
  //Content consists of information about food item
  FoodItemContent content = 2;
  ...
```

Produces `JSON` for Swagger: :
``` 
...       
"content": {
   "$ref": "#/definitions/whisk.api.foodlist.v2.FoodItemContent",
   "description": "Content consists of information about food item",
   "title": "content"
},
...
```

### Annotation based

In annotation based-way we deal with annotations for each type of property. It is more detailed way - you can describe almost everything. 
But this one is more verbose so should be used in the certain cases where we need more details to be provided.

#### Message specs

We are using grpc-annotations to let gen-swagger know how to deal with model in the Swagger Documentation. 
You will start with `option (grpc.gateway.protoc_gen_swagger.options.openapiv2_schema) = {...}` annotation inside a `message` to describe it. 
This annotation includes [fields](https://github.com/grpc-ecosystem/grpc-gateway/blob/master/protoc-gen-swagger/options/openapiv2.proto#L318-L338) to describe a `Scheme`:
- json_schema

    Defines the scheme of `endpoint` or `message`.
     
- descriminator

    Adds support for polymorphism. The discriminator is the schema property name that is used to differentiate between other schema that inherit this schema

- read_only

    Relevant only for Schema "properties" definitions. Declares the property as "read only". This means that it MAY be sent as part of a response but MUST NOT be sent as part of the request. Properties marked as readOnly being true SHOULD NOT be in the required list of the defined schema. Default value is false.
    
- external_docs

    Additional external docs for this scheme.
    
- example

    JSON example of the scheme described

For more details look at [Official specs](https://github.com/OAI/OpenAPI-Specification/blob/3.0.0/versions/2.0.md)

###### <a name="schema"></a> JSON scheme

Fields:
- ref
   
   Ref is used to define an external reference to include in the message.
   This could be a fully qualified proto message reference, and that type must be imported into the protofile. 
   If no message is identified, the Ref will be used verbatim in the output. 
   For example: `ref: ".google.protobuf.Timestamp"`
   
- title
   
   The tittle of the scheme. By default is fully-qualified name.
   
- description
   
   The description of the scheme.
   
- required 
   
   The list of required fields for scheme.

For more details look at [Schema Object specs]([official docs](https://github.com/OAI/OpenAPI-Specification/blob/3.0.0/versions/2.0.md#schemaObject))
   
###### External docs

External docs represents just a link to an external documentation for that entity.

- url 

    The URL for the target documentation. Value MUST be in the format of a URL.
   
- description 
    
    A short description of the target documentation. GFM syntax can be used for rich text representation.
    
For more details look at [External Documentation Object specs](https://github.com/OAI/OpenAPI-Specification/blob/3.0.0/versions/2.0.md#external-documentation-object)

##### Field description 

For a field description you will use the same JSON Scheme but with annotation `[(grpc.gateway.protoc_gen_swagger.options.openapiv2_field) = {...}]`
For more details you could check [this](https://github.com/grpc-ecosystem/grpc-gateway/blob/master/protoc-gen-swagger/options/openapiv2.proto#L127-L171):

##### Example of message scheme

In `.proto` file:
```
message FoodItem {
  option (grpc.gateway.protoc_gen_swagger.options.openapiv2_schema) = {
    json_schema: {
      description: "Some description about food item"
      required: [
        "id",
        "content",
        "order"
      ]

    }
    external_docs: {
      url: "https://google.com"
      description: "Just google it"
    }
    example: {
      value: '{"id": "123", "content":"...", "order": 1}'
    }
  };

  string id = 1
  [(grpc.gateway.protoc_gen_swagger.options.openapiv2_field) = {
      description: "The unique identifier of Food item."
  }];

  FoodItemContent content = 2
  [(grpc.gateway.protoc_gen_swagger.options.openapiv2_field) = {
    description: "The content of the Feed item."
  }];

  int32 order = 3
  [(grpc.gateway.protoc_gen_swagger.options.openapiv2_field) = {
    description: "An order in the resulting list."
  }];
}
```

Produces `JSON` for Swagger::
```
"whisk.api.foodlist.v2.FoodItem": {
      "type": "object",
      "example": {
        "id": "123",
        "content": "...",
        "order": 1
      },
      "properties": {
        "id": {
          "type": "string",
          "description": "The unique identifier of Food item."
        },
        "content": {
          "$ref": "#/definitions/whisk.api.foodlist.v2.FoodItemContent",
          "description": "The content of the Feed item."
        },
        "order": {
          "type": "integer",
          "format": "int32",
          "description": "An order int the resulting list."
        }
      },
      "description": "Some description about food item",
      "externalDocs": {
        "description": "Just google it",
        "url": "https://google.com"
      },
      "required": [
        "id",
        "content",
        "order"
      ]
    },
```


### API specs

API endpoint spec starts in service `rpc` method with annotation `option (grpc.gateway.protoc_gen_swagger.options.openapiv2_operation) = {...}`.
The scheme behind annotation is like [following](option (grpc.gateway.protoc_gen_swagger.options.openapiv2_operation))

- tags

    A list of tags for API documentation control. Tags can be used for logical
    grouping of operations by resources or any other qualified name. If you omit the field
    the name of service will be Used.

- summary
     
    A short summary of what the operation does. For maximum readability in the
    swagger-ui, this field SHOULD be less than 120 characters.

- operation_id

    Unique string used to identify the operation. The id MUST be unique among
    all operations described in the API. Tools and libraries MAY use the
    operationId to uniquely identify an operation, therefore, it is recommended
    to follow common programming naming conventions.
    
- responses ([Described below](#responses))
    
- deprecated
    
    Declares this operation to be deprecated. Usage of the declared operation
    should be refrained. Default value is false.
    
- security ([Described below](#security_req))
    
    A declaration of which security schemes are applied for this operation. The
    list of values describes alternative security schemes that can be used
    (that is, there is a logical OR between the security requirements). This
    definition overrides any declared top-level security. To remove a top-level
    security declaration, an empty array can be used.


#### <a name="responses"></a> Responses

Responses are represented by list of key value pairs `[key: "200", value: {} ]` 

- key 
    
  The key is the HTTP Status Code of response
  
- value

  Value is object that represents the response model which is referenced to `message`;
  ```
    {
      description: "OK"
      schema: {
         json_schema: {
            ref: ".whisk.api.foodlist.v2.FoodItem"
         }
      }
    }
  ```
  Here should be the description of the response and it [schema](#schema) (we should use `ref` to link appropriate scheme)
    
There could be a few values of responses like `[{key: "200", value: {...}}, {key: "400", value: {...}]`

#### <a name="security_req"></a> Security requirements

- security_requirement

  Each name must correspond to a security scheme which is declared in
  the Security Definitions. If the security scheme is of type "oauth2",
  then the value is a list of scope names required for the execution.
  For other security scheme types, the array MUST be empty.

For more details look at [Security requirements specs](https://github.com/OAI/OpenAPI-Specification/blob/3.0.0/versions/2.0.md#securityRequirementObject)

#### Example of API docs

In `.proto` file:
```
service FoodListAPI {
  rpc GetFoodList(GetFoodListRequest) returns (GetFoodListResponse) {
    option (google.api.http) = {
      get: "/foodlist/v2"
    };

    option (grpc.gateway.protoc_gen_swagger.options.openapiv2_operation) = {
              summary: "Get food list."
              responses: [{
                key: "200"
                value: {
                  description: "OK"
                  schema: {
                    json_schema: {
                      ref: ".whisk.api.foodlist.v2.GetFoodListResponse"
                    }
                  }
                }
              }, {
                key: "400"
                value: {
                  description: "Not found"
                }
              }]
              security: {
                security_requirement: {
                  key: "token",
                  value: {
                    scope: [
                      "read:food"
                    ]
                  }
                }
              }
            };
  }
```

Produces `JSON` for Swagger:
```
 "/foodlist/v2": {
      "get": {
        "summary": "Get food list.",
        "operationId": "FoodListAPI_GetFoodList",
        "responses": {
          "200": {
            "description": "OK",
            "schema": {
              "$ref": "#/definitions/whisk.api.foodlist.v2.GetFoodListResponse"
            }
          },
          "400": {
            "description": "Not found",
            "schema": {}
          },
          "default": {
            "description": "An unexpected error response",
            "schema": {
              "$ref": "#/definitions/grpc.gateway.runtime.Error"
            }
          }
        },
        "tags": [
          "FoodListAPI"
        ],
        "security": [
          {
            "token": [
              "some-scope"
            ]
          }
        ]
      }
    },
```

### Common spec for service

To describe the information that should be provided across whole service we use `option (grpc.gateway.protoc_gen_swagger.options.openapiv2_swagger) = {...}`. 
This is a file-based option which goes in the root of file and describes the common fields:

- responses (as it described in API spec)
- securities (as it described in API spec)

Those fields will be added to all endpoints of service, if there doesn't present one. In case there is already exists field with a key it won't be overwriten.

#### An example

In `.proto` file:
```
  option (grpc.gateway.protoc_gen_swagger.options.openapiv2_swagger) = {
    responses: [{
      key: "400"
      value: {
        description: "Bad request error. Specific error codes can be found in endpoint description";
        examples: {key: "application/json", value: '{"error_code": "REAL_CODES_ARE_IN_ENDPOINT_DESCRIPTION","message":"Additional details about error are not static and can be changed"}'};
      }
    },
      {
        key: "401"
        value: {
          description: "Auth error, possible codes: `auth.tokenNotFound` `auth.tokenExpired` `auth.tokenInvalid` `auth.tokenRequired`";
          examples: [{key: "application/json", value: '{"code":"auth.tokenNotFound"}'}];
        }
      },
      {
        key: "500"
        value: {
          description: "This is unexpected response, something is wrong on our side, please contact: help@whisk.com";
        }
      }]
  };

  ...

  
  rpc GetFoodList(GetFoodListRequest) returns (GetFoodListResponse) {
    option (google.api.http) = {
    
    ...   

    option (grpc.gateway.protoc_gen_swagger.options.openapiv2_operation) = {
      responses: [
        {
          key: "400"
          value: {
            description: "Not found"
          }
        }
      ]
    };
  }
```

Produces `JSON` for Swagger:
```
...
  "400": {
    "description": "Not found",
    "schema": {}
  },
  "403": {
    "description": "Returned when the user does not have permission to access the resource.",
    "schema": {}
  },
...
```

## Common example

In `.proto` file: 
```
  option (grpc.gateway.protoc_gen_swagger.options.openapiv2_swagger) = {
    responses: {
      key: "403"
      value: {
        description: "Returned when the user does not have permission to access the resource."
      }
    }
    responses: {
      key: "400"
      value: {
        description: "General not found response."
      }
    }
  };

  ...

  // GetFoodList
  //
  // Get foodlist for user
  rpc GetFoodList(GetFoodListRequest) returns (GetFoodListResponse) {
    option (google.api.http) = {
      get: "/foodlist/v2"
    };
    option (grpc.gateway.protoc_gen_swagger.options.openapiv2_operation) = {
      responses: [
        {
          key: "400"
          value: {
            description: "Not found"
          }
        }
      ]
    };
  }
```

Produces `JSON` for Swagger:

```
...
"paths": {
    "/foodlist/v2": {
      "get": {
        "summary": "GetFoodList",
        "description": "Get foodlist for user",
        "operationId": "FoodListAPI_GetFoodList",
        "responses": {
          "200": {
            "description": "A successful response.",
            "schema": {
              "$ref": "#/definitions/whisk.api.foodlist.v2.GetFoodListResponse"
            }
          },
          "400": {
            "description": "Not found",
            "schema": {}
          },
          "403": {
            "description": "Returned when the user does not have permission to access the resource.",
            "schema": {}
          },
          "default": {
            "description": "An unexpected error response",
            "schema": {
              "$ref": "#/definitions/grpc.gateway.runtime.Error"
            }
          }
        },
        "tags": [
          "FoodListAPI"
        ]
      }
    },
...
```