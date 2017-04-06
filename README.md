# lua-schema-validation

lua-schema-validation is a validation library for Lua.

## About

This library helps to validate data by providing different validators which can be combined in a schema.

Validators can be combined in any way, just be creative about it :) and feel free to contribute.

## Examples

```lua
local v = require "validation"


-- create a validator ...
local foo = v.is_string()

-- and validate data against it
local valid, err = foo("bar")
-- valid = true, err = nil


-- restricted set of values
local choice = v.in_list{"yes", "not sure", "no"}

local valid, err = choice("yes")
-- valid = true, err = nil


-- array of values
local numbers = v.is_array(v.is_number())

local valid, err = numbers{-2, 0, 5}
-- valid = true, err = nil


-- schema/tables combining other validators
local schema = v.is_table{
  foo = v.is_string(),
  choice = v.in_list{"yes", "not sure", "no"},
  numbers = v.is_array(v.is_number())
}

local valid, err = schema{
  foo = "bar",
  choice = "yes",
  numbers = {-1, 0, 1}
}
-- valid = true, err = nil


-- use meta validator, for conditional validation:
--  * validate on certain condition: assert
--  * validate using validator a or b: or_op
--  * optional
local schema = v.is_table{
  type = v.in_list{"a", "b", "c"},
  value = v.assert("type", "a", v.is_integer()),
  flag = v.or_op(v.is_integer(), v.is_boolean()),
  not_needed = v.optional(v.is_string())
}

local valid, err = schema{
  type = "a",
  value = 42,
  flag = true
}
-- valid = true, err = nil


-- nested tables and arrays
local figure = v.is_table{
  details = v.is_table{
    name = v.is_string(),
    description = v.is_string(),
  },
  coordinates = v.is_array(v.is_table{ x = v.is_number(), y = v.is_number()}),
}

local valid, err = figure{
  details = {
    name = "triangle",
    description = "polygon with three edges and three vertices"
  },
  coordinates = {
    { x = -2, y = -1 },
    { x = 0, y = 3 },
    { x = 1, y = -2 }
  }
}
-- valid = true, err = nil


-- all combined together
local schema = v.is_table{
  title = v.is_string(),
  type = v.in_list{"article", "page"},
  category = v.assert("type", "article", v.is_string()),
  rank = v.optional(v.is_integer()),
  details = v.is_table{
    author = v.is_string(),
    status = v.is_boolean()
  },
  tags = v.is_array(v.is_string())
}

local valid, err = schema{
  title = "Hello world",
  type = "article",
  category = "news",
  details = {
    author = "bob",
    status = false
  },
  tags = {
    "hello",
    "world"
  }
}
-- valid = true, err = nil


-- When data does not match against the schema, err describe every error in the schema.
local valid, err = schema{
  type = 24,
  rank = "first",
  details = {
    author = "bob",
    status = 1
  }
}

-- valid = fasle
-- err  = {
--   tags = {
--     "is missing and should be an array."
--   },
--   type = {
--     "is not in list [ 'article' 'page' ]."
--   },
--   title = "is missing and should be a string.",
--   rank = "is not an integer.",
--   details = {
--     status = "is not a boolean."
--   }
-- }
```

## Installation

Install using `luarocks`.

```bash
luarocks install lua-schema-validation
```

## Validators

Validators are used to verify your data. They can be used independently or combined together in a schema representation.

List of available validators:

* `is_string()`                : verify that value is of type `string`
* `is_number()`                : verify that value is of type `number`
* `is_integer()`               : verify that value is of type `number` and is an integer
* `is_boolean()`               : verify that value is of type `boolean`
* `in_list(list)`              : verify that value is from the given list
* `is_table(schema, tolerant)` : verify that value is of type `table` and validate the schema inside it
* `is_array(validator)`        : verify that value is of type `table` and validate every value inside with the given validator

Meta validators:

* `optional(validator)`             : if key is found run the given validator
* `assert(key, value, validator)`   : if data found at key equal value the validator is applied
* `or_op(validator_a, validator_b)` : data is considered valid when validator_a or validator_b return true

## Validate

Validation is performed by calling the function returned by the validator and passing it the data to be validated.

It return a boolean telling you whether the validation is successfull or not. If not it also return a error table structured as the schema and containing error for every faulty key.
