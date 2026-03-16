The Tools menu displays all tools registered with reverge by its collectors. When a collector connects to reverge, it reports the list of tools it has available. This menu allows you to configure default parameters, set API keys, adjust tool scan order, and manage tool classification settings.
<br>
<br>
<center>
<img src="../../assets/tools.png" alt="Tools" width="750" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>
To update a tool's configuration, click on the tool row in the **Tools** Dialog table. The **Tool Details** and **Wordlists** dialog will be displayed for the selected tool. If a tool requires an API key to operate, enter it here. The **Wordlists** dialog allows you to select what wordlists should be made available to the tool during scanning. After making changes, click the **Save** button.
<br>
<br>
<center>
<img src="../../assets/tool_details.png" alt="Tools" width="750" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>
<br>
<br>

## Modules

Below the Tool Details and Wordlists panels, the **Modules** table lists all modules registered under the selected tool. Modules represent the individual scan tasks or checks a tool can perform. The table shows each module's **Name**, **Parameters**, and **Description**.

New modules can be added using the **Add** button in the Modules panel header, and existing modules can be removed by selecting them and clicking the **Delete** button. Modules are invoked automatically during scanning based on the tool's configuration and scan order.
