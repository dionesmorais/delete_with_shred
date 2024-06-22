#### Documentation: Adding "Shred with shred" Option to Caja

##### Objective
Add an option to the Caja context menu to securely delete files using the `shred` command, with actions logged to a system file.

##### Prerequisites
- **Caja-Actions Configuration Tool**
- **shred**
- **Administrative permissions**

##### Steps

###### 1. Install Necessary Tools
Ensure `caja-actions` and `shred` are installed:

```sh
sudo apt-get install caja-actions shred
```

###### 2. Create the Script `delete_with_shred.sh`
Create the script that Caja will call to execute `shred` on the selected file:

```sh
sudo nano /usr/local/bin/delete_with_shred.sh
```

Add the following content to the script:

```sh
#!/bin/bash

# Check if a file was passed as an argument
if [ -z "$1" ]; then
    echo "No file specified." >&2
    exit 1
fi

# Log file
LOGFILE="/var/log/shred.log"

# Get the file name and user
FILENAME="$1"
USER=$(whoami)
DATE=$(date '+%Y-%m-%d %H:%M:%S')

# Execute shred with the specified parameters
shred -z -u -n 5 "$FILENAME"

# Check if shred was successful
if [ $? -eq 0 ]; then
    # Log the action
    echo "$DATE - $USER - $FILENAME was successfully deleted using shred." >> "$LOGFILE"
else
    # Log the failure
    echo "$DATE - $USER - Failed to delete $FILENAME with shred." >> "$LOGFILE"
fi
```

Save and exit the editor (`Ctrl+X`, `Y`, `Enter`).

###### 3. Set Script Permissions

Ensure the script has execution permissions:

```sh
sudo chmod +x /usr/local/bin/delete_with_shred.sh
```

###### 4. Manually Create the Log File

Create the log file and set the appropriate permissions:

```sh
sudo touch /var/log/shred.log
sudo chmod 666 /var/log/shred.log
```

###### 5. Configure the Caja-Actions Configuration Tool

Open the Caja-Actions Configuration Tool:

```sh
caja-actions-config-tool
```

1. Click on "Define a new action".
2. In the "Action" tab:
   - **Context label**: Enter "Shred with shred".
   - **Tooltip**: Enter "Securely delete the file with shred".
   - **Icon**: Optionally, choose an icon for the action.
3. In the "Command" tab:
   - **Path**: Enter `/usr/local/bin/delete_with_shred.sh`.
   - **Parameters**: Enter `%f`.
4. In the "Conditions" tab:
   - **Mimetypes**: Add `all/allfiles`.
5. In the "Context" tab:
   - Ensure the "Display action in location context menus" option is checked.
   - Uncheck "Display action in submenu" to ensure the action appears directly in the context menu.

Save the new action.

###### 6. Restart Caja

Restart the Caja file manager to apply the changes:

```sh
caja -q
caja &
```

###### Testing the New Action

Now, when you right-click a file in Caja, you should see the "Shred with shred" option. Selecting this option will securely delete the file using `shred` and log the action to `/var/log/shred.log`.

##### Example Log Entry

The log file `/var/log/shred.log` will have entries like these:

```
2024-06-22 14:30:00 - user - /path/to/file.txt was successfully deleted using shred.
2024-06-22 14:35:00 - user - Failed to delete /path/to/file.txt with shred.
```

##### Summary of Steps

1. **Install `caja-actions` and `shred`**:
   ```sh
   sudo apt-get install caja-actions shred
   ```
2. **Create the script `delete_with_shred.sh`**:
   ```sh
   sudo nano /usr/local/bin/delete_with_shred.sh
   ```
3. **Set script permissions**:
   ```sh
   sudo chmod +x /usr/local/bin/delete_with_shred.sh
   ```
4. **Create the log file**:
   ```sh
   sudo touch /var/log/shred.log
   sudo chmod 666 /var/log/shred.log
   ```
5. **Configure the Caja-Actions Configuration Tool**:
   ```sh
   caja-actions-config-tool
   ```
6. **Restart Caja**:
   ```sh
   caja -q
   caja &
   ```

This setup provides a clear and accessible way to track all deletion operations performed with `shred` via the Caja context menu.
