# Step 4 - updating the system and securing it

Working in IT, one of my bread and butter tasks will be to keep the system updated, to ensure that I maintain a secure system. 

For this, I used the sudo apt update 
![sudo apt update](/images/step4-list-of-packages.png)
Once I fetched all the latest packages, I installed them all using the sudo apt upgrade command with the -y at the end so that I wouldn't have to continue to input yes.
![sudo apt upgrade](/images/step4-sudo-apt-upgrade--y.png)
Then I used the command sudo apt autoremove -y to remove old, unused packages and free up space.
![autoremove](/images/step4-sudo-apt-autoremove--y.png)
## step 4.2 securing my system

The time has come to set up my firewall. For this I am going to be using the uncomplicated FireWall (UFW). My goal when setting this up is the "Default-deny posture."
I am only going to keep port 22 open, because it is the only one that need to use. My goal is to reduce the attack surface of my server. 

![UFW enabled](images/step4-UFW-enabled.png)
## step 4.3 check system health

To make sure that everything is working correctly with my server, I ran a series of commands that will let me know that my system is healthy and functional.

![system health check](/images/step4-system-health.png)

