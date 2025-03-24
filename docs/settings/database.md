The Database menu on the Settings page can be used to backup and restore the reverge database. The primary use case for this functionality is transferring the data from one instance of reverge to another, typically when upgrading.

## Export Tables
To export the reverge database tables, Click the <img src="../../assets/db_export.png" alt="Export button" width="30"> button at the top right of the **Database Operations** table. This action will queue an export job that will be added to the table. When the job is complete, the database export can be downloaded by clicking the <img src="../../assets/download_btn.png" alt="Download button" width="30"> button. The export file is a compressed tar archive containing reverge data in csv format and the exported table schemas.
<br>
<br>
<center>
<img src="../../assets/export_db.png" alt="Export DB" width="750" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>

## Import Tables
The import process is the mirror operation to the database export feature and expects a compressed tar archive that was produced by performing an **Export**. <span style="color: red;">**It is highly recommmended to perform this operation immediately after installing a new reverge instance to avoid data inconsistency**.</span> To being an import, click the <img src="../../assets/db_import.png" alt="Import button" width="30"> button. In the popup dialog, click the **Choose File** button and select a previously exported tar.gz reverge database export file. Next, click the **Confirm** button.
<center>
<br>
<br>
<img src="../../assets/import_db.png" alt="Import DB" width="450" style="box-shadow: 5px 5px 15px rgba(0, 0, 0, 0.3);">
</center>