## LAB: App Dev: Setting up a Development Environment v1.1

## Overview

In this lab, you will provision a Google Compute Engine virtual machine and install software libraries for Node.js software development on Google Cloud Platform.

## Objectives

In this lab, you will learn how to perform the following tasks:

    - Provision a Google Compute Engine instance.

    - Connect to the instance using SSH.

    - Install software on the instance.

    - Verify the software installation.

## Task 1: Creating a Compute Engine Virtual Machine Instance

        gcloud compute instances create dev-instance --image-family=rhel-8 --image-project=rhel-cloud --zone=us-central1-a

## Task 2: Install software on the VM instance

    - In the SSH session, to update the Debian package list, execute the following command:

            sudo apt-get update

    - To install Git, execute the following command:

            sudo apt-get install git

    - To download the Node.js setup script, execute the following command:

            curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -

    - To install npm and Node.js, execute the following command:

            sudo apt install nodejs

## Task 3: Configure the VM to Run Application Software

In this section, you will verify the software installation and run some sample codes.

    - To check the version of Node.js, execute the following command:

            node -v

    - To clone the class repository, execute the following command:

            git clone https://github.com/GoogleCloudPlatform/training-data-analyst

    - To change the working directory, execute the following command:

            cd ~/training-data-analyst/courses/developingapps/nodejs/devenv/

    - To run a simple web server, execute the following command:

            sudo node server/app.js

    - Return to the Cloud Console VM instances list, and click on the External IP address for the dev-instance.

            You should see a Hello GCP dev! message from Node.js.

    - Return to the SSH window, and stop the application by pressing Ctrl+C.

    - To install the Node.js library for Compute Engine, execute the following command:

            npm install

    - To run a simple Node.js application that lists Compute Engine instances, execute the following command:

            node list-gce-instances.js

    