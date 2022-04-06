# AUX PROJECT 1: SHELL SCRIPTING

Create the project folder called Shell and move into it.
`mkdir Shell && cd Shell`

Create a csv file name names.csv
`touch names.csv`

Open the names.csv file
`vim names.csv`
and make a list of names in it.

create the key files
`touch id_rsa.pub id_rsa onboardingusers.sh`

then move the files into the server directory called Shell

![Copy](https://github.com/femie15/darey/blob/main/project%201/aux-project/1-copy%20files%20to%20server.PNG)

![Copy](https://github.com/femie15/darey/blob/main/project%201/aux-project/2-viewfiles.PNG)

create user group named "developers"  `sudo groupadd developers`

Run the onboardingusers.sh file with `sudo ./onboardingusers.sh` for linux and mac OR use `sudo bash ./onboardingusers.sh` for windows system

![Copy](https://github.com/femie15/darey/blob/main/project%201/aux-project/3-run.PNG)

There was a mistake in the permission script i did, so i'm rewriting the script to only give permissions to the "authorized_keys" users file

![Copy](https://github.com/femie15/darey/blob/main/project%201/aux-project/3-runb.PNG)

View all addes users.

![Copy](https://github.com/femie15/darey/blob/main/project%201/aux-project/4-usersAndGroup.PNG)

Login as one of the users and try to perform some "sudo-level" command. it should be rejected.
![user login](https://github.com/femie15/darey/blob/main/project%201/aux-project/5-loginAsUser.PNG)
