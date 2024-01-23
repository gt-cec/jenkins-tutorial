# Jenkins (CI/CD) Tutorial

This tutorial outlines installing Jenkins and setting up a CI/CD pipeline for Windows through GitHub. This is useful for our studies from which we host our own servers, and CI/CD can automatically deploy from GitHub changes.

## Overview

This tutorial covers the following steps:

1. Set up a GitHub repo for a study.
2. Clone the GitHub repo and set up a study program. In this case, we make a shell script to open a browser.
3. Push the study program to GitHub.
4. Install Jenkins.
6. Set up a Jenkins project to poll GitHub for code updates and pull them.
7. Add a system task to run the shell script.
8. Set up the Jenkins project to run the system task after pulling from GitHub.

## Setting up GitHub

If you have not already, set up [SSH keys on GitHub](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent). This is needed for authentication, and you will need your private key for Jenkins to authenticate with GitHub. You will need to generate the SSH keys, add the SSH key to your ssh-agent, and add the public key to your GitHub account (in Settings, SSH and GPG Keys).

Create a GitHub repository for the project, or use this one. I will use this `jenkins-tutorial` repository as an example for this tutorial.

Once the repository is created, open a terminal (use Powershell on Windows), use `cd` and `dir` to navigate to where you want the repository, and clone the repository. In this case I will place it on the Desktop.

```
cd Desktop
git clone git@github.com:gt-cec/jenkins-tutorial
```

You now have the project repository, and can make edits as you please.

## Make a study script

Next we will make a fake study script. Navigate to the `studies` folder and create a new script.

## Clone the GitHub repo

