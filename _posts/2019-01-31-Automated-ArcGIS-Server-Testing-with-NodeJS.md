---
layout: post
title:  Automated ArcGIS Server Testing with NodeJS
author: Joshua Tanner
description: Generate alerts for ArcGIS Server inconsistencies or failures
image: assets/images/warning-bees.jpg
---

Murphy's Law states that anything that can go wrong with ArcGIS Server will go wrong with ArcGIS Server.  Sometimes this is due to the fault of the software or problems with your server.  Other times it's our own fault when accidentally changing field names or not checking a box to enable a certain capability.  When this happens, it's best to get in front of the issue before your end users notice.  When publishing production services, your users depend on things being not only up, but being consistent to keep their apps running.

For this blog post, we're going to be testing 2 things:

1. ArcGIS Server is accessible (not dead)
2. An ArcGIS Server service has the correct name

To accomplish this task, we will write a series of tests using the REST API testing framework [Frisby.js](https://www.frisbyjs.com/).  Frisby.js is a simplified JavaScript testing framework.  If you're unfamliar with writing tests, don't be afraid.  Tests essentially only have 2 components; an instruction and a expectation.  For example, we might define a test as:

1. INSTRUCTION: Go to http://google.com
2. EXPECTATION: Expect that we get an HTTP response of status code [200](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200), meaning everything is ok

The code for this test would look like this:

```javascript
it ('should return a status of 200', function () {
  return frisby
    .get('http://api.example.com')
    .expect('status', 200);
});
```

Let's get started by setting up our project.

## Setup

+ [nodejs](https://nodejs.org/en/) will need to be installed.  I'm using 10.15.0 at the time of this tutorial.

We will want to create a directory to store our project files.

```bash
$ mkdir test-suite
$ cd test-suite
```

Next, let's initiate a node project with `npm init`.  You can accept all the defaults by presseng `enter`.

```bash
$ npm init
```

Follow [instructions](https://www.frisbyjs.com/installation.html) to install the JavaScript testing framework, Frisby.js.

```bash
$ npm install frisby --save-dev
$ npm install jest --save-dev
```

Create a new folder for storing our tests and then create a new file for writing our test code.

```bash
$ mkdir __tests__
$ touch __tests__/arcgis_server.spec.js
```

Now that everything is setup, let's write our tests in our new file `__tests__/arcgis_server.spec.js`

The first test will test that ArcGIS Server is responding okay (HTTP Status Code 200).

```javascript
const frisby = require('frisby')

it('should resond with a status 200', function () {
  return frisby.get('http://sampleserver1.arcgisonline.com/arcgis/rest/services')
    .expect('status', 200)
    .catch((err) => {
      console.log(err)
    })
})
```

The code above does the following:

1. It loads the testing library Frisby.js with `require('frisby')`
2. Sets up a test to be run, with the title `should resond with a status 200`
3. Makes a HTTP request to ArcGIS Server using `frisby.get()`
4. It defines the expectation that our HTTP request should respond with a `status` `200`
5. Logs out an error if something went wrong

You can run this test from your terminal (need to be in root of project) with [jest](https://jestjs.io/).

```bash
$ ./node_modules/.bin/jest
```

Assuming everything is setup correctly, you should see the following output:

```bash
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        2.108s
Ran all test suites.
```

In the same file `__tests__/arcgis_server.spec.js`, let's add another test.

Test that a specific service is available and is named correctly.

```javascript
it('should have traffic camera layer', function () {
    return frisby.get('http://sampleserver1.arcgisonline.com/ArcGIS/rest/services/Louisville/LOJIC_PublicSafety_Louisville/MapServer/2?f=json')
        .then((res) => {
            expect(JSON.parse(res.body))
                .toHaveProperty('name', 'TrafficCams')
        })
})
```

In this test, we are testing that a specific service in ArcGIS Server is correctly named 'TrafficCams'.  Under normal circumstances, we would use the Frisby.JS assertion `.expect('json', 'name', 'TrafficCams')`.  However, this service that is hosted in ArcGIS Online does not return the proper headers for our code to interpret the response as json.  Instead, the server responds with a HTTP header `Content-Type` = `text/plain`.  To handle this, we need to parse the response into JSON and check the key/value using the [Jest](https://jestjs.io/docs/en/expect#tohavepropertykeypath-value) expect method `.toHaveProperty(key, value)`.


Finally, here is what our final file should look like `./__tests__/arcgis_server.spec.js`:

```javascript
const frisby = require('frisby')

it('should resond with a status 200', function () {
  return frisby.get('http://sampleserver1.arcgisonline.com/arcgis/rest/services')
    .expect('status', 200)
    .catch((err) => {
      console.log(err)
    })
})

it('should have traffic camera layer', function () {
    return frisby.get('http://sampleserver1.arcgisonline.com/ArcGIS/rest/services/Louisville/LOJIC_PublicSafety_Louisville/MapServer/2?f=json')
        .then((res) => {
            expect(JSON.parse(res.body))
                .toHaveProperty('name', 'TrafficCams')
        })
})
```

Navigate to the root of the project and run jest again:

```bash
$ ./node_modules/.bin/jest

PASS  __tests__/arcgis_server.spec.js
✓ should resond with a status 200 (218ms)
✓ should have traffic camera layer (202ms)

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        2.41s
Ran all test suites.
```

## What Next

The next thing is to identify what tests you need to write to make sure your server and services and producing the desired output.  It's important to understand how your users are using your services and what is critical to keep consistent.  Some things to test are:

1. Field Naming
2. Completed Metadata
3. HTTPS (SSL Cert Issues)
4. Capabilities enabled (query, wms, etc)

There are an infinite number of things to test and testing is not limited to just ArcGIS Server.

## Automate

Once you have finished writing tests to test your infrastructure, the next thing to do is automate your workflow.  I typically schedule these tests to run every hour and trigger an alert if a test fails.  Upon failure, I use [nodemailer](https://nodemailer.com/about/) to send myself an email with the test output.  This helps me troubleshoot errors, hopefully before anybody knows about them :).