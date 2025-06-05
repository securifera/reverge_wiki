A Target in reverge represents a collection of CIDR blocks, domains, or specific URLs that can be scanned by a reverge collector. The targets menu provides the ability to manage targets. This includes the addition and removal of targets, defining target scope, and scheduling target scans.
<br>
## Add Target
To create a target in reverge click on the <img src="../../assets/add_btn.png" alt="Save button" width="30">  button in the top right corner of the Targets dialog.
<br>
<br>
<center>
<img src="../../assets/target_menu.png" alt="Target Table" width="700" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>
Enter a name for the target in the **Add Target** dialog and click **Save**.
<br>
<br>
<br>
<center>
<img src="../../assets/add_target.png" alt="Add Target" width="400" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
## Remove Target
To remove a target from reverge, select the checkbox to the left of the target name in the Targets dialog and click on the <img src="../../assets/delete_btn.png" alt="Delete button" width="30">  button in the top right corner.
<br>
<br>
<center>
<img src="../../assets/target_delete.png" alt="Delete Target" width="750" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
## Hosts
After setting up the target, you will be taken to the target dashboard. This dashboard displays key metrics about the target and provides a comprehensive list of all detected hosts.
<br>
<br>
<center>
<img src="../../assets/target_hosts.png" alt="Target Hosts" width="750" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
## Scope
The Scope submenu lets you define the target's scope. Subnets, Domains, and URLs can be added to the scope individually by clicking on the respective <img src="../../assets/add_btn.png" alt="Add button" width="30"> button.
<br>
<br>
<center>
<img src="../../assets/target_scope.png" alt="Target Scope" width="750" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>
Target scope can also be imported in bulk by clicking on the <img src="../../assets/import_btn.png" alt="Import button" width="30"> button. The import feature requires a line-delimited scope file. To proceed, click the **Browse** button to select the scope file, then click **Import**.
<br>
<br>
<br>
<center>
<img src="../../assets/import_scope.png" alt="Import Scope" width="600" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
## Scans
The **Scans** submenu within the **Target** page lets you initiate new scans for a specified scope and view results from past scans. The **Scan Schedule** table lists the currently scheduled scans along with their configurations This includes the selected ports, chosen tools, scan recurrence, and the specified collector.
<br>
<br>
<center>
<img src="../../assets/target_scans.png" alt="Target Scans" width="750" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>
To schedule a scan to be executed on the target, click on the click on the <img src="../../assets/add_btn.png" alt="Add button" width="30"> button  in the **Scan Schedule** dialog or the <img src="../../assets/scan_btn.png" alt="Scan button" width="30"> button on the target dialog menu. The **Network Scan** modal requires multiple parameters be configured before proceeding. First, select a [Collector](/collectors/collectors/) to perform the scan. Then, choose the **Hosts** to scan, which can either be the entire scope or specific IPs or subnets entered manually.. 
<br>
<br>
<center>
<img src="../../assets/scan_dlg.png" alt="Target Scans" width="450" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>
Likewise, you must select the **Ports** to scan. The drop-down list offers several predefined groups, or you can choose **Custom** to manually enter individual ports. Manually entered **Custom** ports can be space or comma delimited and include ranges, e.g. 1-1000.
<br>
<br>
<br>
<center>
<img src="../../assets/port_list.png" alt="Target Scans" width="450" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>
The **Schedule** drop-down list contains options for either a **One Time** scan or setting up a **Recurring** scan by defining the schedule using a cron-like format.
<br>
<br>
<center>
<img src="../../assets/scan_schedule.png" alt="Target Scans" width="450" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>
The **Passive Collection** container includes tools that gather data about a target from third-party sources without any direct interaction with the target network. The **Active Collection** container lists tools that communicate directly with the target.
<br><br>
Each of the selected [Tools](/resources/tools) will run with the default arguments specified in the **Resources**->**Tools** menu. To modify these default arguments for the scan, click on the <img src="../../assets/drop_dwn.png" alt="Add button" width="20" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">  button and enter the new arguments in the tool arguments input box. 
<br>
<br>
<br>
<center>
<img src="../../assets/custom_args.png" alt="Target Scans" width="450" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>
After selecting the scan parameters, click the **Submit** button. This will add an entry to the **Scan Schedule** table.
<br>
<br>