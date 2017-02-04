---
layout: post
title: Unittesting your logstash 5.x filters
date:   2017-02-04
categories:
  - Logstash
  - Testing
---

I was trying to find a simple solution to test my Logstash filter pipeline
but the blogs and documentation seemed to cover older versions and/or focus
on Ruby. No hard feelings against Ruby but we're not a Ruby shop so this got me thinking,
how hard can it be? Well it turns out not hard at all.

The basis of my solution is language and framework agnostic but the examples will use
Node and the Jest test suite.

We'll use the following project structure for the examples

```
| - filters // Logstash filters
  | - 00-test-input.conf
  | - ** // your filters goes here
  | - 99-test-output.conf
| - input // Logstash test input
  | - test1.json
  | - test2.json
| - output // where logstash will store the files
| - tests // Jest test suite
| - docker // files used to build our container
  | - Dockerfile
  | - run-filter.sh
```


##### Docker container #####
Let's start with building the container we will use to do our Logstash processing.
Piping input into the stdin plugin let's us start Logstash and have it shut down
when it's processed all input so there's no need to manage it's lifetime.

```Dockerfile
FROM logstash
COPY run-filter.sh /
VOLUME /test
ENTRYPOINT ["/run-filter.sh"]
```

This is quite basic, we copy our script and set it as the entrypoint so that we can
use the container as a regular command.

The script we're using is really simple and could possibly be specified directly
in the Dockerfile but I wasn't sure about the pipe and didn't want to spend time testing.

```sh
#!/bin/bash
cat /test/input/*.json | /docker-entrypoint.sh -f /test/filters/
```

docker-entrypoint.sh is provided by the logstash container so we just call that in the same way
while piping our input into it.

Build like so `docker build -t paxxi/logstash-test .`

Once we have built this container we can invoke it like `docker run -it --rm -v $PWD:/test paxxi/logstash-test`

##### Logstash filters #####
Our container isn't all that useful yet, it will happily pipe whatever we have in our `input` folder
into Logstash but there's no output yet.

Let's fix this with a test input config and a test output config. We'll start with the input

```
input {
  stdin { codec => json }
}
```

Simple enough, we expect json piped to stdin, __remember to have a blank line at the end of each json file__.

Next up our output config

```
output {
  file {
    codec => json
    path => "/test/output/%{[test_name]}"
  }
}
```

Also quite straight forward, we write each event to a file named after the key `test_name` in our inputs.
Probably a bit of a hack but it lets us easily separate each test result into it's own file for later processing.

##### Test inputs #####
How you decide to gather your test inputs is up to you. I decided to output a few events from filebeat and then use
that JSON as a template and replacing the `log` key with the different messages.

##### The actual tests #####
We finally have all the pieces in place to start working on the tests, that's what we came for :)

Here I'm using Nodejs and [Jest](https://facebook.github.io/jest/) but any language that can read JSON can be
used.

A simple test can look like this

```javascript
const fs = require('fs');

let result;
beforeAll(() => result = JSON.parse(fs.readFileSync('../output/test1.json')));

test('service_name should be set and correct', () => {
  expect(result.service_name).toBe("traefik");
});
```

And that's all there's to it, now we have a test pipeline that we can build upon
and extend.

##### End result #####
We can run our tests with two steps (obviously this can be integrated but personally it's running in Jenkins so this is good enough)

```
docker run -it --rm -v $PWD:/test paxxi/logstash-test
cd tests && jest
```

Happy testing!