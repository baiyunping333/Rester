Rester  + YAML + Exec + ... + More to come.
=======

This is a *heavily modified fork* of the very good
https://github.com/chitamoor/Rester project, currently under heavy
development. If you're a web platform developer, this is (will be) a
great framework for declarative testing.


Framework for testing (RESTful) APIs
----------------------------------
Rester allows you to test your APIs using a non-programatic or non-GUI
based approach (which are some of the more conventional ways of
testing RESTFul APIs). *Rester* is inspired by various unit testing
frameworks like JUnit, 'unittest' (python) etc and is conceptually
organized like those frameworks but is geared towards testing RESTful
API endpoints. With *Rester*, all tests are specified in YAML or JSON,
so it can also be used by non-programmers as well.


#So, why Rester?
Testing RESTful APIs generally involves two prediticable steps -

- Invoke the API end point
- Validate the response - headers, payload etc

Most testing tools available for testing RESTful APIs use some sort of
a GUI based approach which doesn't lend itself towards re-use, better
code organization, abstraction etc and some of the other benefits that
are generally available with more programmatic frameworks like
JUnit. Programmatically building test cases provides the highest level
of flexibility and sophistication, but the downside to this approach
is that it ends up with lots of fairly tedious and repetitive
code. Conceptually, Rester is similar to existing unit testing
frameworks, but it uses JSON (instead of a programming language) to
implement/specify the actual tests. Programmers and non-programmers
can reap the benefits with the Rester approach.

Note: As of now Rester only supports APIs that don't require explicit
authentication of calls, but future versions will support
OAuth. Rester was mainly created to test internal APIs that generally
bypass the need for authentication of the calls.


#Practical uses of Rester
- Perform "integration" testing of internal and external API endpoints
- Examine and test complex response payloads
- You can simply use Rester to dump and analyze API responses - headers, payload etc.

#Assumptions
- Rester does not manage the life-cycle of the container or the server
  that exposes the API endpoints, but assumes the API endpoints (to be
  tested) are up and avaliable.
- Unlike other unittesting frameworks however, Rester does guarantee
  the order of execution of the **TestSteps** within a
  **TestCase**. For a better understanding of TestSteps and TestCases
  see the "General Concepts" section below. The **ordering** will come
  in handy if you want to test a series of API end-points (invoked in
  succession) that modify system state in a particular way.

#General Concepts

* **TestSuite**:
 A *TestSuite* is collection of *TestCases*. The idea is to group
 related 'test cases' together. Use either YAML or JSON.

Globals can be defined in the either a *TestSuite* or *TestCase*.

```
globals:
  variables:
    request_opts: # special for the request library.
      verify: false # disable CERT checking for SSL
    http: "http://localhost:8905"
    https: "https://localhost:8081"
test_cases:
  - servers.yaml
  - cors.yaml
  - xdomain.yaml
  - notfound.yaml
```

* **TestCase**:
 A *TestCase* contains one or more *TestSteps*. You can declare
 **globals** variables, which are scoped to the *TestCase*, and add to
 or override the **globals** defined in the *TestSuite*.

```
name: "Ping"
globals: 
  variables: 
    http: "http://localhost:8905"
    https: "https://localhost:8081"
testSteps:
 - ...
 - ...
```

* **TestStep**:
All of the action takes place in a **TestStep**.

A TestStep contains the following:

- **API end point invocation** - As part of the API endpoint invocation, you can provide the following params -
  - URL
  - HTTP headers
  - URL params
  - HTTP method - get, put, post, delete, patch - any method supported
    by the *requests* library. ('get' is used by default)

URL is the only mandatory param.

- A series of assert statements specified as part of an **AssertMap**

Example of a TestStep (JSON):

```
  testSteps: [
    {
       "name":"Name of TestStep",
  		   "apiUrl":"http://example/api/v1/helloworld/print",
       "assertMap":{
             "headers":{
                 "content-type":"application/json; charset=utf-8"
             }
             "payLoad":{
                "message":"Hello World!"
             }
       }
    }
  ]
```

A complete example as YAML, leveraging the *yaml references*:

```
name: "Ping"
globals: 
  variables: 
    http: "http://localhost:8905"
    https: "https://localhost:8081"
testSteps: 
  - &test1
    name: "ping http"
    apiUrl: "{http}/ping"
    method: "get"
    params:
      __raw__: true
    assertMap: 
      headers: 
        connection: "keep-alive"
        content-type: "text/plain; charset=utf-8"
      payLoad:
        __raw__: "pong"
  - 
    <<: *test1
    name: "ping https"
    apiUrl: "{https}/ping"
```

#Example Output

See: https://gist.github.com/ninowalker/1fe8aad019feab3fe265

#Installation

This fork (which adds YAML and exec support):
 `pip install git+https://github.com/ninowalker/Rester.git@master`

#Rester command line options
- Run the default test case -

  `apirunner`

  This will look for the default test case, ***test_case.json in the current directory***
- Run a specific test case
  use the command line option ***--tc=<file_name>***

  e.g. invoke a test case specified in the file "./rester/examples/test_case.json"

  `apirunner --tc=./rester/examples/test_case.json`

- Run a specific test suite
  use the command line option ***--ts=<file_name>***

  e.g. invoke a test suite specified in the file "./rester/examples/test_suite.json"

  `apirunner --ts=./rester/examples/test_suite.json`

#Other command line options
- Adjust the log output or details
  Rester support varying levels of logs - DEBUG, INFO, WARN, ERROR. You can
  specify the level using the command line option ***--log=<LEVEL>***

  e.g. run the API with INFO level

  `apirunner  --log=INFO`

- Just dump the JSON output

#TestCase options
- **Skipping tests**

#TestStep options

#Examples of API request invocations
- Specify the HTTP headers as part of an API request
 ```
  testSteps: [
    {
       "name":"Name of TestStep",
  		   "apiUrl":"http://example/api/v1/helloworld/print",
  		   "headers":{
            "content-type":"application/json;"
       },
       ....
    }
  ]
  ```

- Specify the URL params as part of an API request.
  There are two ways to specific URL params, which are mentioned below -

  ```
  testSteps: [
    {
       "name":"Name of TestStep",
  		   "apiUrl":"http://example/api/v1/helloworld/print",
  		   "headers":{
  		      ...
       },
       "params":{
            "param_1":"value1",
            "param_2":"value2"
       },
       ....
    }
  ]
  ```


   ```
  testSteps: [
    {
      "name":"Name of TestStep",
  		"apiUrl":"http://example/api/v1/helloworld/print?param_1=value1&param_2=value2",
       ....
    }
  ]
  ```

- Perform an HTTP POST
  ```
  testSteps: [
    {
        "name":"Name of TestStep",
        "apiUrl":"http://example/api/v1/helloworld/print",
        "headers":{
            ...
        },
        "method":"post"
        "params":{
            "param_1":"value1",
            "param_2":"value2"
        },
       ....
    }
  ]
  ```

- Get a non-JSON response, using the *__raw__* parameter.
```
name: "Ping"
globals: 
  variables: 
    http: "http://localhost:8905"
testSteps: 
 - name: "ping http"
    apiUrl: "{http}/ping"
    method: "get"
    params:
      __raw__: true # this is a magic parameter
    assertMap: 
      headers: 
        content-type: "text/plain; charset=utf-8"
      payLoad:
        __raw__: "pong"
```

#Examples of assert statements
As mentioned previously, all of the assert statements are specified within an **assertMap** element

- Assert "content-type" HTTP header
 ```
  testSteps: [
    {
      "name":"Name of TestStep",
      "apiUrl":"http://example/api/v1/helloworld/print?param_1=value1&param_2=value2",
       ....
    }

    "assertMap":{

        "headers":{
          "content-type":"application/json; charset=utf-8"
        },

        ....

    }
  ]

- Assert specific payload elements -
  "output.level" is 2
  "output.result" is eqal to "Message Success"
  "output.status" is greater than 3
  "output.body" contains the word 'launched'

 ```
  testSteps: [
    {
      "name":"Name of TestStep",
      "apiUrl":"http://example/api/v1/helloworld/print?param_1=value1&param_2=value2",
       ....
    }

    "assertMap":{
        "headers": {
           ....
        },

        "payLoad":{
            "output.level":2,
            "output.result":"Message Success",
            "output.status":"-gt 3",
			"output.body":"exec 'launched' in value"
        },
        ....

    }
  ]
  ```

# Assert logical operators:

- **-gt** - greater than

  ```

  e.g. parent.child > 3

      "payLoad":{
            "parent.child":"-gt 3",
      }
  ```

- **-ge** - greater than eqal to

  ```
  e.g. parent.child >= 3

      "payLoad":{
            "parent.child":"-ge 3",
      }
  ```

- **-lt** - lesser than

  ```

  e.g. parent.child < 2

      "payLoad":{
            "parent.child":"-lt 2",
      }
  ```

- **-le** - lesser than eqal to

  ```

  e.g. parent.child <= 2

      "payLoad":{
            "parent.child":"-le 2",
      }
  ```

- **-ne**  - not eqal to

  ```

  e.g. parent.child.message != "success"

      "payLoad":{
            "parent.child.message":"-ne success",
      }
  ```

- **-eq**  -  equal to

```
  e.g. parent.child.message == "error"
      "payLoad":{
            "parent.child.message":"-eq success",  # either will work
            "parent.child.message":"success",
      }
  ```

- **exec**  -  evaluate a python expression where the node is passed
  in as *value*

```
  e.g. parent.child.message == "hello world"
      "payLoad":{
            "parent.child.message":"exec len(value) > 7 and value.endswith('world')",
      }
  ```


# Basic JSON Type checking
## The following JSON types are supported - Integer, Float, String, Array, Boolean

-  check if parent.child.message is a String
```
       "payLoad":{
            "parent.child.message":"String",
      }
```

- check if parent.child.version is an Integer
```
      "payLoad":{
            "parent.child.version":"Integer",
      }
```

- check if parent.child is an Object
```
      "payLoad":{
            "parent.child":"Object",
      }
  ```


# Using variables declarations
- Variables are declared in the "globals" section of the TestSuite

  ```
   "globals":{
      "variables":{
        "baseApiUrl":"https://example.com",
        "api_key":"YOUR_KEY",
        "rule_key":"CONFIG_KEY"
      }
   },
   ...
  testSteps: [
    {
      "name":"Name of TestStep",
      "apiUrl":"http://{baseApiUrl}/api/v1/helloworld/print?param_1=value1
      ....
    }
  ]
  ```

# Contact
rajeev@chitamoor.com

This fork: nino@livefyre.com

#TODO
- Use meta-programming to allow direct integration into unittest
  frameworks, and run with tests a la `nose`.
- Switch `assertMap` to `asserts`, so that you can have multiple
  asserts on a single key.
- Use code `eval` for all tests; `value == '123'` is barely longer than
  `123`.
- Allow module imports for inclusion in the `eval` tests.
- Support assignments into the variable name space, to enable
  continuity of values between tests. E.g. a `POST` returns an `id`
  which is used in the next step `GET`.
- Merge variables into the eval space; no string expansion on asserts.
- Support for enums
- Support for OAuth
- Run in `record mode` to capture responses for testing directly.

# Explored and passed
- Look at https://pypi.python.org/pypi/mongoql-conv/0.4.1 for building
  python expressions.
  -Ultimately, substituing python for a subset will hamper the project.
