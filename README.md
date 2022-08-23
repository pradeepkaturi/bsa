==================================================
		BUILD SHEET AUTOMATION - README
==================================================
DESCRIPTION

Build Sheet Automation is a Shell Script which can pull the VM Info from the Cloud using Cloud CLI commands and fetch OS Level details for each IP from the extracted Cloud Data.
Results of Build Sheet VM Info will be written to the vm_info.txt in pipe seperated text for each field in the Build Sheet.

PRE-REQUISITES

1. Master_Os_level_script.sh should be available in the present working directory
2. User Os Level Password should be same across the landscape
3. NCPA Plugin(/usr/lib64/nagios/plugins/check_ncpa.py) and NCPA Token required for to fetch windows Os level details

#Development Version: 1.0 , 04/14/2022
	
	- Removed Nagios verification function as we are having separate Nagios audit shell script for Nagios verification. No need to provide list of ip’s/host names in nagios_config.txt
	- Modified scritpt to not to send sending Os_level_details.sh to target servers for to get OS/DB/Application details. Build_Sheet_Automation.sh itself will run the Os_level_details.sh in target Linux machines and gets data.
	- Modified Script to get the AWS region automatically using 'curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone'

#Development Version: 1.1 , 04/20/2022

	- Updated logic not to store any passwords while executing Os_level_details.sh in target servers
	- Added feature to fetch VM Info for specific list of IP's provided in the file and file name should be give as argument to Build_Sheet_Automation.sh
	- Updated SSH Commands in Build_Sheet_Automation.sh to use -e for passing passwords as an arguments
	- Added feature to verify NCPA Plugin available or not in the patch sever, If not script will print Cannot NCPA [$HOSTNAME: NCPA Plugin Not Available] for each IP.
	
#Development Version: 1.2 , 05/02/2022

	- Removed Hardcoded NCPA Token and modified script to prompt for NCPA token while execution. From now user has to provide NCPA Token manually while execution.
	- Added base64 Encoding and Decoding process for User Password and NCPA Token
	- Changed Command for to get Last OS Patch from 'rpm -qa --last' to 'rpm -q --last kernel' as per Curtis Hoffman suggestion
	
#Development Version 1.3, 07/15/2022 (Updated script to use on Azure as well)
	
	- All the execution steps/procedure was same like earlier(i.e. for AWS) where script can fetch cloud and VM info for complete servers in cloud and also for the specific list of IP’s provided in file and file name as an argument to the script
	- Additionally Script will prompt for the user input of Cloud (AWS or AZURE). If user input other than AZURE or AWS then script will exit after asking again for one more time
	- If user input is AZURE script will fetch the Azure cloud info for VM list, Network and NIC list for each Resource Group, VM Sizes specified for each location
	- Script will parse cloud data for each IP Address and fetches OS level Info

#Development Version 1.4, 07/26/2022 (Updated script to create BSA TMP folders while script exec and output to VM_INFO_"$DT_BSA".txt”)	

    - Create Build_Sheet_Automation in $HOME folder of the user
    - Create TMP folder under Build_Sheet_Automation  with name BSA_tmp_{YMD_HM}
    - All tmp files will be placed in BSA_tmp folder and TMP_DIR will be removed at the end of the script
    - Output will be written to VM_INFO_{YMD_HM}.txt under Build_Sheet_Automation
    - Changed “< Os_level_details.sh” to “< $ANSIBLE_HOME/dba_ansible/playbooks/Os_level_details.sh” in Build_Sheet_Automation.sh file.
    - Changed “cat vm_info.txt | tail -14” to “cat $HOME/Build_Sheet_Automation/VM_INFO_"$DT_BSA".txt” in Build_Sheet_Automation.sh file.

	
#Development Version 1.5, 08/19/2022 (Removed TAGS SID and Added FETCH OS DETAILS, APP SID, DB SID and DB HOSTNAME)

    - Removed SID which we are getting from TAGS at Cloud level as we noticed incorrect information/Not updated information
    - Added new Columns APP SID,DB SID and DB Hostname which we will get from OS level
    - Added new Column “Fetch OS Details” in between Cloud Level and OS Level Details to inform fetching OS level details was successful or Cannot SSH/NCPA due to Access or Instance stopped state
    - Fixed NCPA plugin compatibility issues with python versions by carrying the check_ncpa.py(supports v3) to the working directory (/etc/ansible/dba_ansible/playbooks) along with Os_level_details.sh
