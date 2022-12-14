# How to discover VMX features using GCP VM

1. Setup gcloud account 

2. Loign to console

3. Create a VM 
   ( for this project i created with following features 
    machine type: n2-standard-2
    OS: Ubntu
    )
4. Connect to machine using either by Browser based SSH or your own machine

5. Install gcloud CLI
  <br> https://cloud.google.com/sdk/docs/install

6. Once intsalled initialize gcloud CLI
   <br> `gcloud init`

7. We need to enable nested virtualization inorder to discover VMX features

8. To enable nested virtualization
  <br> https://cloud.google.com/compute/docs/instances/nested-virtualization/enabling

9. In our case we already have a VM so we will export the property of the instance using
   - `gcloud compute instances export instance-1  --destination=export.yaml   --zone=us-west4-b`
   - Add following text to export.yaml file if its not already there 
     ``` 
     advancedMachineFeatures:
        enableNestedVirtualization: true
     ```    

   - Update the VM with following command
     <br> `gcloud compute instances update-from-file instance-1  --source=export.yaml   --most-disruptive-allowed-action=RESTART --zone=us-west4-b`

10. To check if nested virtualization is enabled, connect to VM and run the following command,it should any number other than 0
    <br> `grep -cw vmx /proc/cpuinfo`

11. Create a directory and add the files Makefile and .c file to find the features

12. But first install dependencies
    <br>`sudo bash`
    <br>`apt install gcc make`
    <br>`exit`
    <br>`sudo apt-get install linux-headers-$(uname -r) `

13. If some issue with gcc then run update command and install again
   <br> `update apt-get`
   <br>`apt install gcc`
   <br> `apt install make`

14. Run make to create output inside the directory
    <br>`make`

15. Once you see the autogenerated files to see the output load the .ko file in kernel and read
     <br>`sudo insmod cmpe283-1.ko`
     <br>`dmesg`

16. To find about the MSR and controls that you want query 
    <br>https://www.intel.com/content/dam/develop/public/us/en/documents/325384-sdm-vol-3abcd.pdf

17. For this project i have queried following 6 MSRs
     - IA32_VMX_PINBASED_CTLS (5 controls)
     - IA32_VMX_PROCBASED_CTLS (22 controls)
     - IA32_VMX_PROCBASED_CTLS2 (27 controls)
     - IA32_VMX_EXIT_CTLS (16 controls)
     - IA32_VMX_ENTRY_CTLS (13 controls)
     - IA32_VMX_PROCBASED_CTLS3  (4 controls)

18. Some observations about the output
    - In primary procbased controls in my Vm the ability to set ???Activate Secondary Controls??? was true and for ???Activate Tertiary Controls??? was false.
    - In tertiary procbased control the ability to set for all 4 controls were false.