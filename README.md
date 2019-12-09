# aws-ssm-to-env

> Parse AWS Systems Manager parameters to environment variables.

## Table of Contents

  * [About](#about)
  * [Usage](#usage)
     * [Parameters](#parameters)
     * [Examples](#examples)
        * [String values](#string-values)
        * [Custom prefix](#custom-prefix)
        * [Simple JSON parameter values](#simple-json-parameter-values)
        * [Complex JSON values](#complex-json-values)
  * [TODO](#todo)

## About

This action is designed to read AWS SSM parameters and exports them as environmental variables.

Script can parse string value parameters as well as parameters with stringified JSON values. For simple JSON objects
a shortcut parameter `simple_json` can be used to convert all key-values from JSON into environmental variables.

**Important note:** Although the AWS CLI command in this Action is very simple, Action is using preview version of
AWS CLI (version 2) which is still not recommended for production use ([more info](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)).
Author will update commands after future AWS CLI updates if necessary.

## Usage

### Parameters

Parameter name | Type | Required | Default Value | Description
--- | --- | --- | --- | ---
`ssm_parameter` | string | true | | AWS Systems Manager parameter name (path)
`prefix` | string | false | AWS_SSM_ | Custom environmental variables prefix
`simple_json` | boolean | true | false | Parse parameter values as one-level JSON object and convert keys to environmental variables  (see example below).
`jq_params` | string | true | | Custom space-separated [`jq` filters](https://stedolan.github.io/jq/) (see example below).

### Examples

#### String values

Parse simple string value stored in AWS SSM `my_parameter_name` parameter:
```yaml
name: Parse SSM parameter

on:
  push

jobs:
  aws-ssm-to-env:
    runs-on: ubuntu-latest
    steps:
      - name: aws-ssm-to-env
        uses: bomb-on/aws-ssm-to-env@master
        env:
          AWS_REGION: ${{ secrets.AWS_REGION }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        with:
          parameter_name: 'my_parameter_name'
```

Example above will set environmental variable `AWS_SSM_MY_PARAMETER_NAME` with value from the AWS SSM parameter itself.

#### Custom prefix

Parse simple string value stored in AWS SSM `my_parameter_name` parameter and export environmental variable with 
custom prefix:
```yaml
name: Parse SSM parameter

on:
  push

jobs:
  aws-ssm-to-env:
    runs-on: ubuntu-latest
    steps:
      - name: aws-ssm-to-env
        uses: bomb-on/aws-ssm-to-env@master
        env:
          AWS_REGION: ${{ secrets.AWS_REGION }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        with:
          parameter_name: 'my_parameter_name'
          prefix: FOO_
```

Example above will set environmental variable `FOO_MY_PARAMETER_NAME` with value from the AWS SSM parameter itself.

#### Simple JSON parameter values

Parse simple one-level JSON object and create environmental variables from all keys:
```yaml
name: Parse JSON SSM parameter

on:
  push

jobs:
  aws-ssm-to-env:
    runs-on: ubuntu-latest
    steps:
      - name: aws-ssm-to-env
        uses: bomb-on/aws-ssm-to-env@master
        env:
          AWS_REGION: ${{ secrets.AWS_REGION }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        with:
          parameter_name: 'my_json_parameter'
          simple_json: true
```

If `my_json_parameter` in the example above is a JSON string like
```json
{"foo": "bar", "baz": 1}
```
environmental variables will be set as:
```sh
AWS_SSM_FOO=bar
AWS_SSM_BAZ=1
```

#### Complex JSON values

Pass a custom, space-separated filter(s) to `jq` and parse desired parts of JSON object:
```yaml
name: Parse JSON SSM parameter

on:
  push

jobs:
  aws-ssm-to-env:
    runs-on: ubuntu-latest
    steps:
      - name: aws-ssm-to-env
        uses: bomb-on/aws-ssm-to-env@master
        env:
          AWS_REGION: ${{ secrets.AWS_REGION }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        with:
          parameter_name: 'my_json_parameter'
          jq_filter: '.db[]|select(.default).host .db[]|select(.default).port'
          prefix: DB_
```

If `my_json_parameter` in the example above was a JSON string like
```json
{"db": [{"host": "my.db.host.com", "port": 1337, "default": true}, {"host": "other.host", "port": 42}]}
```
environmental variables will be set as:
```sh
DB_HOST=my.db.host.com
DB_PORT=1337
```

## TODO

 - [ ] Use official Docker container once it becomes available (https://github.com/aws/aws-cli/issues/3291, https://github.com/aws/aws-cli/issues/4685)
 - [ ] Write tests (https://github.com/kward/shunit2)
