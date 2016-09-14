---
title: "How To Test REST API with API Blueprint and Dredd"
excerpt: "Glue together your API Documentation and the backend"
layout: post
date: 2013-10-17 18:00:00 +0100
author: netmilk
published: YES
---


As a follow up for [my previous blogpost][] which is introducing [Dredd][]—the tool for testing APIs from API Blueprint, I've prepared a simple [example application][dredd-example] in [Express.js][] to demonstrate how to write a blueprint, a backend, test it and let it be tested forever in the CI. 

Note I'm using Express.js for the backend but Dredd, API Blueprint and Apiary do not depend on any specific programming language. Dredd interacts with the backend app only through the HTTP layer. That means the process as described below can be applied on any backend platform and framework in any language. 

Let's move on – it's super-easy!

### 1. Write a blueprint

Start thinking about your API, create an Apiary project with a sample blueprint and save it. You can use [my example blueprint][blueprint]. Do not forget to invite others to join the discussion. Enjoy generated [documentation][] and play with the [mock][].

### 2. Link with GitHub

- Create the GitHub project
- Connect Apiary with GitHub on the setting tab
- Clone the repo locally

### 3. Prepare the Backend Application

- Swich to your local repo directory
- Install [Node.js][]
- Bootstrap application environment

        npm install express coffee-script mongous

- Create empty Express application in file `app.coffee`

        express = require 'express'
        app = express()
        app.listen 3000
        console.log('Listening on port 3000');


### 4. Make it Red

- Add a test script to `scripts/test`

        #!/bin/sh
        ./node_modules/coffee-script/bin/coffee app.coffee &
        sleep 5
        PID=$!
        dredd apiary.apib http://localhost:3000/
        RESULT=$?
        kill -9 $PID
        exit $RESULT


- Install Dredd globally 

        npm install -g dredd

- See the test fail

        $ ./scripts/test
        $ echo $?
        1
  
### 5. Make it Green

- Write the code of the backend application
- Run the test again and see it pass

        $ ./scripts/test
        $ echo $?
        0
        
### 6. Make it Eternal

And finally – the holy grail! When you plug it into a Continuous Integration system any change of the API Blueprint in your GitHub repository will update the API documentation generated by Apiary and run the test harness. At this point you have always up-to-date API documentation. And what is even better – it is absolutely free of charge!

- Sign in to [Travis-CI] Continuous integration service
- Find your project in the list of your GitHub repositories and enable it
- Add a Travis dotfile to `.travis.yml` with:

        language: node_js
        node_js:
        - 0.8
        - 0.10
        before_install:
        - npm install -g dredd
        script: ./scripts/test
        services:
        - mongodb
        before_script:
        - mongo mydb_test --eval 'db.addUser("test", "test");'

- Commit and push your work to GitHub
- Wait for the build and proudly place the status badge ( ![Build Status](https://travis-ci.org/apiaryio/dredd-example.png?branch=master) ) to the [Readme][] of your project and to the API [documentation][]

We strongly appreciate any feedback in Dredd's GitHub [issues] or directly through our [support].

## Supported Continuous Integration Services

Dredd works with several Continuous Integration Services. I've tested it with [Travis-CI][], [Circle-CI][], [Codeship][], [Semaphore][], [Teamcity][], [Jenkins][], [Atlassian Bamboo][] and probably many others. 

![Travis-CI](http://blog.apiary.io/images/ci-systems/travis.gif?2)
![Circle-CI](http://blog.apiary.io/images/ci-systems/circle.gif?2)
![Codeship](http://blog.apiary.io/images/ci-systems/codeship.gif?2)
![Semaphore](http://blog.apiary.io/images/ci-systems/semaphore1.gif?2)
![Team city](http://blog.apiary.io/images/ci-systems/teamcity.gif?2)
![Jenkins](http://blog.apiary.io/images/ci-systems/jenkins.gif?2)
![Atlassian Bamboo](http://blog.apiary.io/images/ci-systems/bamboo.gif?2)

[Apiary]: http://apiary.io
[my previous blogpost]: http://blog.apiary.io/2013/10/10/No-more-outdated-API-documentation/
[Readme]: https://github.com/apiaryio/dredd-example/blob/master/README.md
[blueprint]: https://gist.github.com/netmilk/6885268
[documentation]: http://docs.dreddexample.apiary.io/
[mock]: http://docs.dreddexample.apiary.io/traffic
[dredd-example]: http://github.com/apiaryio/dredd-example
[Express.js]: http://expressjs.com/
[issues]: https://github.com/apiaryio/dredd-example/issues
[support]: mailto:support@apiary.io
[Dredd]: https://github.com/apiaryio/dredd
[API Blueprint]: https://apiblueprint.org/
[Travis-CI]: https://travis-ci.org/
[Codeship]: https://www.codeship.io/
[Teamcity]: http://www.jetbrains.com/teamcity/
[Circle-CI]: https://circleci.com/
[Jenkins]: http://jenkins-ci.org/
[Hudson]: http://hudson-ci.org/
[Atlassian Bamboo]: https://www.atlassian.com/software/bamboo
[Semaphore]: https://semaphoreapp.com/
[Node.js]: http://nodejs.org/
[NPM]: https://npmjs.org/
[NVM]: https://github.com/creationix/nvm