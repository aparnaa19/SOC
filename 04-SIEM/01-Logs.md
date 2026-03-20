## Logs 

- multiple devices in a network communicate with one another. 
- these activities are generated as logs 
- the devices from which the logs are generated are called as log sources. 
- these logs are extremely helpful for identifying malicious activities or general troubleshooting

----

# Types of Logs 

logs are generally divided into 2 categories:
- host centric 
	- A user accessing a file
	- A user attempting to authenticate
	- A process execution activity
	- A process adding/editing/deleting a registry key or value
	- PowerShell execution
- network centric
	- SSH connection
	- A file being accessed via FTP
	- Web traffic
	- A user accessing the company's resources through VPN
	- Network file sharing Activity

----

# Problem with logs 

- Numerous Log Sources 
	- examining the logs on each device one by one in case of an incident can be tedious.

- No Centralization 
	- As logs reside on the machines on which they are generated,  you may need to connect with each log source via SSH, RDP, etc., to analyze logs from multiple log sources
	- This is very inefficient and can waste a lot of your valuable time during the investigations

- Limited Context 
	- Individual logs cannot tell the whole story of an activity. 
 	- For instance, you observed a file access event in a system, which is generally normal activity. 
	- However, if you correlate different log sources, you might come to know that this file was accessed by a user who accessed this machine through lateral movement after compromising another machine 

- Huge number of logs 
	- Format Issues Different log sources generate logs in various formats. 
	- Analysts need to know all these formats to analyze them, which can be extremely difficult
