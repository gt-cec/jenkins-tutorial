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

Next we will make an example study script. In your terminal, navigate to the `studies` folder and create a new script, just use your name. This tutorial is made for you to actually do these steps.

```
cd studies
notepad jack.bat
```

This created a shell script file called `jack.bat` and opened it in Notepad for editing. If you want, you can open Explorer to see that the project folder and files exist.

For our example study, our shell script will simply open a browser to the Georgia Tech homepage. In reality you can do a whole lot more with this script (run Python scripts, compile code, launch browsers on different screen, run other programs, and so on). Copy the below in:

```
echo "Now starting the user study!"
# Open Microsoft Edge
msedge gatech.edu
echo "All set up!"
# Keep the terminal open until the ENTER key is pressed
read
```

In your terminal, test your script:

```
jack.bat
```

With our study script ready, track the changes, commit them, and push to GitHub. In the terminal:

```
git add jack.bat
git commit -m "added a study script"
git push 
```

If your SSH keys are set up, and you have access to the CEC GitHub, the code changes will push without issue.

At this point you have made a shell script to launch your study, and you have pushed it to GitHub. In practice this repository is the project repository, so to do this for another project you would just write a shell script there.

## Install Jenkins

Jenkins is a tool to manage CI/CD functionality. It has a pretty UI, is beginner friendly, and is widely used across industry. Jenkins is designed for both smaller home projects (what we do) and larger distributed projects (like having dedicated test and production servers that queue builds and test code functionality).

Jenkins is written in Java, which means there are two ways to install or run Jenkins:

1. Through their installer, this creates a system service that launches Jenkins in the background on login. It is possible that GT does not allow for using the installer.
2. Through running their `.jar` file, this runs Jenkins as the user and requires that [Java is installed](https://openjdk.org/).

If the lab computers allow it, I would use the installer. If not, the `.jar` works perfectly fine. You can download either from the [Jenkins website](https://jenkins.io).

Jenkins does not have a window interface, instead it runs a webserver (which is useful for running it on dedicated servers that do not have a desktop). Once Jenkins is running, open a browser to `localhost:8080`, and you will start a setup process. Just follow the instructions, and the default settings should be fine.

For setting up, our projects only need the most basic functions. When you reach the "Plugins" page, deselect everything with the "None" button at the top, and select `git`, `GitHub`, and `MS Build` (if your project requires compiling a Visual Studio solution).

