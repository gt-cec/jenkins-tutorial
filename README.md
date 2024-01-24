# Jenkins (CI/CD) Tutorial

This tutorial outlines installing Jenkins and setting up a Continuous Integration / Continuous Deployment pipeline for Windows through GitHub. This is useful for our studies from which we host our own servers, and CI/CD can automatically deploy from GitHub changes.

## Overview

This tutorial covers the following steps:

1. Set up a GitHub repo for a study.
2. Clone the GitHub repo and set up a study program. In this case, we make a shell script (`.bat` script) to open a browser.
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

Next we will make an example shell script that represents the steps required to launch our user study. Open Powershell, navigate to the `studies` folder and create a new script, but use your name instead of `jack`. This tutorial is made for you to actually do these steps.

```
# navigate to the studies/ folder in the tutorial
cd jenkins-tutorial
cd studies
# launch Notepad to create a file, use your name, in my case jack.bat
notepad jack.bat
```

This created a shell script file called `jack.bat` and opened it in Notepad for editing. If you want, you can open Explorer to see that the project folder and files exist. Copy the below into your shell script:

```
echo "Now starting the user study!"
start msedge gatech.edu
echo "All set up!"
```

For our example study, our shell script will simply open a browser to the Georgia Tech homepage. In practice you can do a whole lot more with this script - run Python scripts, compile code, launch browsers on different screen, run other programs, and so on. You can close Notepad now.

In Powershell, test your script. It should briefly open a terminal window and launch Microsoft Edge.

```
.\jack.bat
```

If Microsoft Edge does not open, check your spelling.

Our study script is ready, so track the changes, commit them, and push to GitHub. In the terminal:

```
# make sure you are in the jenkins-tutorial/studies/ folder
cd studies
# add your <name>.bat file, in my case, jack.bat
git add jack.bat
# commit the file to git, remember to specify a commit message
git commit -m "added a study script"
# push the changed to GitHub
git push 
```

If your SSH keys are set up correctly, and your GitHub account have access to the CEC GitHub, the code changes will push without issue.

At this point you have made a shell script to launch your study, and you have pushed your shell script to GitHub. This repository represents the project repository, so to do this for another project you would just write a shell script there.

## Install Jenkins

Jenkins is a tool to manage Continuous Integration / Continuous Deployment functionality. It has a pretty UI, is beginner friendly, and is widely used across industry. Jenkins is designed for smaller home projects (what we do) as well as larger distributed projects (like having dedicated test and production servers that queue builds and test code functionality).

There are two ways to install Jenkins:

1. Through their Windows installer, this creates a system service that launches Jenkins in the background on login. You do not need to have Java installed for this. We found that the GT computers cannot install Jenkins without admin privileges.
2. Through running the Generic Java (`.jar` or `.war`) file, this runs Jenkins as the user and works with GT computers. This requires that [Java is installed](https://oracle.com/java/technologies/downloads) -- on the GT computers, open Software Center and install Amazon Coretto (a Java equivalent).

I will assume you are using a lab computer, the main difference is how you start Jenkins (with the installer, it starts automatically).

To run the `.jar` (or `.war` on Windows), open Powershell and navigate to where the `.jar` is located, probably `Downloads`. Run the following in Powershell:

```
# navigate to Downloads
cd ~/Downloads
# check if Java is installed
java -version
# run Jenkins
java -jar .\Jenkins.war
```

You will see some logging in the terminal. Jenkins does not have a window interface or desktop program, instead it runs its own webserver. To view or make changes to Jenkins, you access the webserver through the port 8080. Open a new tab in your web browser, navigate to `localhost:8080`, and you will start the setup process.

1. In the Secret Password page: follow the instructions on the page (open a file and copy/paste the contents), this verifies that you are the one accessing Jenkins.
2. In the Plugins page: Select "Customize", deselect all plugins by selecting "None" at the top, and then select the `git`, `GitHub`, and `MS Build` (if your project compiles a Visual Studio solution) plugins. We don't need anything more for our projects.
3. In the Create Admin User page: this account is only for you to access Jenkins, so use `CEC` as the username and make a password you can share with labmates.

You should now see the Jenkins dashboard. Well done so far!

If you are using a GT computer, Git is installed however for some reason GT does not include the path to `git.exe` in its [Windows PATH variable](https://en.wikipedia.org/wiki/PATH_(variable)). This means Jenkins cannot access `git.exe`, so it also cannot access your GitHub repositories. Since we are using the Java version, we can instead add the `git.exe` path to the Path environment variable in the Powershell instance that we run Jenkins with. If you can access Git from Powershell (e.g., running `git.exe` does not throw an error), you can skip this step since it means `git` is already in your PATH environment variable.

1. In the Powershell instance that is running Jenkins, use `control-c` to exit Jenkins.
2. Run `$Env:Path = "C:\Program Files\Git\bin;$Env:Path"`, this adds the location of `git.exe` to the Path variable.
3. Run Jenkins again, with `java -jar jenkins.war`.

If you'd like, you can also make this into a shell script, for example `jenkins.bat`:

```
$Env:Path = "C:\Program Files\Git\bin;$Env:Path"
java -jar jenkins.war
```

With Jenkins installed, set up, and able to access `git`, we can now make our first job.

## Make Your First Job

A "Job" is a CI/CD configuration. In our case we want to make a job that regularly polls GitHub for changes and relaunches our server setup script (jack.bat). This is a relatively simple job.

**General**: Add a brief description, and keep the other settings unchecked. "GitHub Project" does not mean "GitHub Repository", a Project is a separate GitHub feature you can use to help manage your project.

**Source Code Management**: Select "Git". For the Repository URL, use the SSH-format of the project URL, in this case:

```
git@github.com:gt-cec/jenkins-tutorial
```

For the credentials, you will need to enter your SSH private key so Jenkins can use it to authenticate as you. Click "Add" -> "Jenkins", set the Kind to "SSH Username with private key", leave the text fields blank, check "Enter directly", and click "Add". Open the private SSH key (probably `C:\Users\YOUR_USER\.ssh\id_ed25519`) in Notepad, copy the entire contents (including the blank line at the end), and paste it into this Private Key field. If you set a Passphrase for your SSH key, enter it in the Passphrase text field.

![image](https://github.com/gt-cec/jenkins-tutorial/assets/23145260/01346ff2-8558-4af3-97e3-92c38c283ef6)

Change the "Branches to build" text field from `*/master` to `*/main`, which is what GitHub uses as their primary branch name. If you want to deploy from a different branch, set that here.

The other settings can be default, if your project uses Git LFS you can set that in "Additional Behaviors".

**Build Triggers**: Select "Poll SCM", this will have Jenkins periodically run `git pull` to update. In a proper system you would use GitHub Actions to tell Jenkins when a build has happened, however since our Jenkins servers are not public we have to instead use polling. For the schedule, use `* * * * *`, which will poll every minute. The stars probably make no sense -- this scheduling format is called a [Cron expressions](https://crontab.cronhub.io/).

Save the job, and open the "Git Polling Log". Within a minute you should see it populate, if there are error message you should address them now.

* If the log says that `git.exe` was not found, revisit setting the `Path` variable and testing that the Powershell instance that runs Jenkins is able to run `git.exe`.
* If the log mentions access issues or complains that the repository was not found, open up the job's "Configure" and check that you wrote the repository name correctly, and that your credentials are selected.

If the panel below "Build History" shows the latest build with a green checkmark, it means your Git setup is configured properly.

## Running the Study Shell Script

At this point you have a project on GitHub that Jenkins is cloning and regularly pulling to. Now you need to tell Jenkins to run the study shell script you made (`<name>.bat`) after it pulls a new commit. We will do this through a "Build Step" in the job configuration.

![image](https://github.com/gt-cec/jenkins-tutorial/assets/23145260/e0bf0eea-6e93-4e85-9272-7966d3c5877d)

After Jenkins pulls your code changed, it creates a build process that runs all the Build Steps you specify, in order. This means that if your build creates subprocesses, all the subprocesses will be terminated when the build process ends. For example, if your project runs a "detached" Python script, the Python script will exit when the build steps are complete. Some programs (like Microsoft Edge) create themselves as new processes instead of as subprocesses, so are not terminated. Since we usually want our projects to be "forever processes" after they build and start, there are two ways we can handle this:

### Persistent Processes: The Hacky Way

The intuitive option is to ignore the problem and block the build process from ending so the build (and its subprocesses) runs forever. This can be done by adding a permanent "while loop" at the end of your shell script, such as by running a Python script that never exits. This is bad practice but it technically works.

For this tutorial, Microsoft Edge runs as its own process, so there are no changes needed to the study shell script. If your study uses multiple Python scripts, try something like this for your study shell script:

```
echo Launching the study!
# start the first Python script in detached mode (the &)
python scripts/script1.py &
# start the loop script, say main.py is a server that never exits
python scripts/main.py
echo Exiting!
```

This is frowned upon because the point of a build is for it to run and finish, not to hang in perpetuity, so this approach prevents any Post-build Steps or subsequent Build Steps from running.

### Persistent Processes: The Proper Way

The proper approach is to make a system task that runs your study's shell script, so that the script is created as a separate process. We then configure the Jenkins build to tell Windows (or linux) to run the task.

On Windows, make the system task by following these steps. On linux, look up how to do this for your distribution.

1. Open Task Scheduler
2. Click "Create Task...", name it whatever you want, for this example I will call it "Study Server"

![image](https://github.com/gt-cec/jenkins-tutorial/assets/23145260/9a016c7e-14c2-4a90-aad0-1b0f22f6b03d)

3. In the "Triggers" tab, you can add a trigger to make the task run when the user logs in, although I would skip this for the tutorial.

![image](https://github.com/gt-cec/jenkins-tutorial/assets/23145260/254e3c07-34c9-46bf-b0a7-8838418231ed)

3. In the "Actions" tab, create a new action of type "Start a program", and set the target to the location of the study shell script _in the Jenkins workspace_ (i.e., `"C:\Users\YOUR_USER\.jenkins\workspace\Study Test\studies\jack.bat"`). This means that the task will launch from Jenkins' saved code. Wrap the path in quotes if it has spaces, so the action does not try to execute something else.

![image](https://github.com/gt-cec/jenkins-tutorial/assets/23145260/2f3ee02a-071e-47e2-9cd1-80d0466cc6f1)

4. In the "Settings" tab, uncheck the "Stop the task if it runs longer than" box (so our study code can run permanently), and change the "If the task is already running..." selector to "Stop the existing instance" (so the task kills the previous study instance).

![image](https://github.com/gt-cec/jenkins-tutorial/assets/23145260/0bf1cf87-102d-4db7-9874-c42a1ce10ef6)

5. Press "Ok" and the task will be created. Test it by right-clicking the task and clicking "run". If you followed the steps, Microsoft Edge will open to the Georgia Tech homepage. If nothing happens, go back and make sure you have followed the steps correctly.

### Setting the Build Step

Returning to Jenkins, go to the bottom of the job's configuration and click `Build Step -> Execute Windows batch command`. On linux, choose `Execute shell`. The text field here is where you will enter a new shell script that gets executed at this Build Step. This can be multiple lines, however I recommend not using this field to replace your study shell script as this Build Step script is not saved to Git.

If you are using the hacky way, use this field to specify the relative path of the study shell script, so the script is launched:

```
.\studies\jack.bat
```

If you are using the proper way, use this script to start the "Study Server" task you made, which launches the study shell script:

```
# for Windows
schtasks /run /tn "Study Server"
```

![image](https://github.com/gt-cec/jenkins-tutorial/assets/23145260/3686aeb9-28b7-4528-bee9-8833b44a1fcb)

And with that, you are all set! Save the job at the bottom of the page, go back to the job's dashboard, and test the build process by clicking "New Build". You should see the build appear in the middle-left, and a few seconds later, see Microsoft Edge open to the Georgia Tech homepage. As usual, if Microsoft Edge does not open, review your steps at this time.

## Testing the Auto-Deploy

At this point Jenkins is all set up: you have your project repository on GitHub, you made a job on Jenkins that pulls from that repository, you created a system task to launch the study shell script, and you are starting that task when Jenkins pulls code changes. That whole process is not automatic, which should save you some time. All that is left is to test the system end-to-end.

1. Open the `studies/jack.bat` file from your Desktop (not Jenkins' version). This represents the codebase you would have on your personal device.
2. Change the URL that Microsoft Edge opens to something else, say, `https://github.com/gt-cec`.
3. `add`, `commit`, and `push` the changes following the process we used previously.
4. In a few minutes, when your job next syncs with GitHub, you should see a new Microsoft Edge instance or tab open to the CEC GitHub page.

At this point, congratulations! You are now using CI/CD. That is the end of this tutorial.

## Jack's Notes

### Launching Jenkins on Login.

If you are setting up Jenkins on a computer that you plan to use for persistent studies, you may want Jenkins to start when you log in. Move the `jenkins.war` file to somewhere out of your way, make the `jenkins.bat` script to set the environment path, and create a system task to run `jenkins.bat` on log in. This is pretty much what the Jenkins installer does.

### Logs

To be safe, you should not store user study logs within the Jenkins workspace, since there is a risk of those logs being overwritten if the workspace is deleted. Instead, the proper form is to use a command-line argument from the Task Scheduler task to specify the folder where you want to the logs to be stored, and using that argument in your code to determine the path ([Python example](https://stackoverflow.com/questions/4033723)).

### Settings

It is worthwhile to scroll through the entire job configuration to see what you want and do not want for your job. Same with the rest of the "Manage Jenkins" settings, everything is nicely documented. You can also play with the Post-build Action "Set GitHub commit status" to see the status and success of your build directly on GitHub -- many use this so you are not left wondering if your build made it through successfully.

### Credentials/Keys with Code

If you have credentials that you do not want posted to GitHub (API keys, user study secrets, login details, etc), the proper method is to set them as Environment Variables that you [access through your code](https://stackoverflow.com/questions/4906977). You can set environment variables through Jenkins through "Manage Jenkins" -> "System" -> "Global Properties" -> check "Environment Variables".

### Custom Shells

If you would like to use a custom shell (for example, Git Bash instead of Powershell), check out "Manage Jenkins" -> "System" -> "Shell". This should alleviate needing to set the Path environment variable before running `jenkins.jar`, however I have not tested it.
