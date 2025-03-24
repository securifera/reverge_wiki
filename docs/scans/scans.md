Scan results can be accessed from two different places in reverge. The main **Dashboard** menu lists scans that have been executed against any target in reverge. The second place is from the **Targets** menu under each specific target. After clicking on a target, click the **Scans** submenu in the top right dialog. To view the details of a scan, click on the corresponding row in the scan table. 

The default page will display basic details about the scan and list all hosts identified. This view will show the IP address for each host,  any domains associated with it, and the number of open ports detected.
<br>
<br>
<center>
<img src="../../assets/scan_page.png" alt="Scan Page" width="750" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>
To access detailed information about each host's ports, click on the corresponding row in the scan table. This will open a detailed view showing all open ports for that host, the components running on them, and any screenshots that have been collected.
<br>
<br>
<br>
<center>
<img src="../../assets/host_expand.png" alt="Host" width="750" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>
## Jobs
If you want details on what scan jobs have been executed in the particular scan, click on the **Jobs** menu at the top right of the scan page. This table will display each tool that was executed, any additional arguments, the target hosts/domains/ports, and the execution status. This table can be exported in JSON format by clicking the <img src="../../assets/download_btn.png" alt="Export button" width="30"> button. To view up-to-date scan logs related to the execution of scan jobs, visit the collector details panel for the associated [Collector](/collectors/collectors/#manage-collector)
<br>
<br>
<center>
<img src="../../assets/scan_jobs.png" alt="Host" width="750" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>
## Filter
If you're viewing a very large scan and need to filter the results, click on the <img src="../../assets/chevron.png" alt="Add button" width="30"> button located in the top-left corner of the screen. This will display a filter menu that provides a mechanism for narrowing down the displayed results according to your specific criteria.
<br>
<br>
To filter the results, select the desired checkboxes and click the **Filter** button.
<br>
<br>
<center>
<img src="../../assets/scan_filter.png" alt="Host" width="650" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>
## Port Details
To view detailed information about each port, click on the corresponding row in the port table. This will open a detailed view listing the components associated with the port, domains extracted from TLS certificates, modules scan results, and HTTP endpoints and their respective screenshots.
<br>
<br>
<center>
<img src="../../assets/port_inst.png" alt="Port" width="750" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>