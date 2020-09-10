## LAB: App Dev: Storing Application Data in Cloud Datastore v1.1

## Overview

In this lab, you will review the case study application, an online Quiz. You will store application data for the Quiz application in Cloud Datastore.

The Quiz application skeleton has already been written for you. You will clone a repository that contains the skeleton using Google Cloud Shell, review the code using the Cloud Shell editor, and view it using the Cloud Shell web preview feature.

Then you will modify the code that stores data to use Cloud Datastore.

## Objectives

In this lab, you will learn how to perform the following tasks:

    - Harness Cloud Shell as your development environment.

    - Integrate Cloud Datastore into a NodeJS application.

## Previewing the Case Study Application

In this section, you will access Cloud Shell, clone the git repository containing the Quiz application, and run the application.

1. Clone source code in Cloud Shell
    
            git clone https://github.com/GoogleCloudPlatform/training-data-analyst

2. Configure and run the case study application

    - To change the working directory, execute the following command:

            cd ~/training-data-analyst/courses/developingapps/nodejs/datastore/start

    - To export an environment variable, GCLOUD_PROJECT that references the GCP Project ID, execute the following command:

            export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID

    - To install the application dependencies, execute the following command:

            npm install

    - To run the application, execute the following command:

            npm start

## Adding Entities to Cloud Datastore

In this section, you will write code to save form data in Cloud Datastore.

1.  Create an App Engine application to provision Cloud Datastore

        - Return to Cloud Shell and stop the application by pressing Ctrl+C.

        - To create an App Engine application in your project, execute the following command in Cloud Shell:

                gcloud app create --region "us-central"

2.  Import and use the NodeJS Datastore module

        - Open the ...gcp/datastore.js file in the Cloud Shell editor.

        - Load the config module from the parent folder.

        - Load the @google-cloud/datastore module.

        - Declare a Datastore client object named ds.

                // TODO: Load the ../config module

                const config = require('../config');

                // END TODO

                // TODO: Load the @google-cloud/datastore module

                const Datastore = require('@google-cloud/datastore');

                // END TODO

                // TODO: Create a Datastore client object, ds
                // The Datastore(...) factory function accepts an options // object which is used to specify which project's
                // Datastore should be used via the projectId property.
                // The projectId is retrieved from the config module. This // module retrieves the project ID from the GCLOUD_PROJECT // environment variable.

                const ds = Datastore({
                projectId: config.get('GCLOUD_PROJECT')
                });

                // END TODO

3. Write code to create a Cloud Datastore entity

        - Declare a constant named kind, initialized with the value 'Question'.

                datastore.js

                // TODO: Declare a constant named kind
                //The Datastore key is the equivalent of a primary key in a // relational database.
                // There are two main ways of writing a key:
                // 1. Specify the kind, and let Datastore generate a unique //    numeric id
                // 2. Specify the kind and a unique string id

                const kind = 'Question';

                // END TODO

        - In the create(...) function, remove the existing Promise.resolve({}) placeholder statement from the create(...) function.

        - Declare a constant called key to store the key for this entity.

        - Declare a constant named entity and initialize it with the key and the quiz question properties extracted from the form data.

        - Use the Datastore client object (ds) to save the entity by calling the save(entity) method.

                datastore.js

                // The create({quiz, author, title, answer1, answer2,
                // answer3, answer4, correctAnswer}) function uses an
                // ECMAScript 2015 destructuring assignment to extract
                // properties from the form data passed to the function

                function create({ quiz, author, title, answer1, answer2,
                                answer3, answer4, correctAnswer }) {
                // TODO: Declare the entity key,
                // with a Datastore generated id

                const key = ds.key(kind);

                // END TODO

                // TODO: Declare the entity object, with the key and data

                const entity = {
                key,
                // The entity's members are represented in a data property.
                // This is an array where each element represents one
                // member in the entity. Each element is an object with a // name and a value
                data: [
                    { name: 'quiz', value: quiz },
                    { name: 'author', value: author },
                    { name: 'title', value: title },
                    { name: 'answer1', value: answer1 },
                    { name: 'answer2', value: answer2 },
                    { name: 'answer3', value: answer3 },
                    { name: 'answer4', value: answer4 },
                    { name: 'correctAnswer', value: correctAnswer },
                ]
                };
                // END TODO

                // TODO: Save the entity, return a promise
                // The ds.save(...) method returns a Promise to the
                // caller, as it runs asynchronously.

                return ds.save(entity);

                // END TODO
                }

4. Run the application and create a Cloud Datastore entity

        - Save the ...gcp/datastore.js file and then return to the Cloud Shell command prompt.

        - To start the application, execute the following command:

                npm start

        - In Cloud Shell, click Web preview > Preview on port 8080 to preview the quiz application.

        - Click Create Question.

        - Complete the form with the following values, and then click Save.

        - Return to the Cloud Platform Console and, on the Navigation menu, click Datastore.

        - On the Datastore page, click Entities.
















