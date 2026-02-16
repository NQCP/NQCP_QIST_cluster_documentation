# Connecting from Windows

This guide covers how to SSH into the QIST HPC cluster and transfer files from a Windows machine.

For general cluster documentation (account creation, partitions, job submission, etc.), see the [main README](../README.md).

## SSH via WSL

If you have Windows Subsystem for Linux installed, you can use SSH exactly as on Linux:

1. Open your WSL terminal (e.g. Ubuntu).
2. Connect:
   ```bash
   ssh <your-username>@fend01.hpc.ku.dk
   ```

> [!TIP]
> You can use any frontend node. See the [main README](../README.md) for available frontends.

### SSH config (optional)

To avoid typing the full hostname every time, add the following to `C:\Users\<you>\.ssh\config` (create the file if it doesn't exist):

```
Host qist
    HostName fend01.hpc.ku.dk
    User <your-username>
```

Then connect with just:

```
ssh qist
```

## SSH via PuTTY

If you prefer a graphical SSH client or are on an older version of Windows:

1. Download PuTTY from <https://www.putty.org>.
2. Open PuTTY and enter:
   - **Host Name:** `fend01.hpc.ku.dk`
   - **Port:** `22`
   - **Connection type:** SSH
3. Click **Open**.
4. Log in with your HPC username and password.

> [!TIP]
> Save the session in PuTTY (enter a name under "Saved Sessions" and click "Save") so you don't have to re-enter the details each time.

## SSH via Windows Terminal / PowerShell

Windows 10 (version 1809+) and Windows 11 include a built-in OpenSSH client. No extra software is needed. Note that there are some issues with the Windows Terminal/Powershell approach (see [#10](/../../issues/10))

1. Open **Windows Terminal** or **PowerShell**.
2. Connect to the cluster:
   ```
   ssh <your-username>@fend01.hpc.ku.dk
   ```
3. On first connection, you will be asked to confirm the host fingerprint — type `yes`.
4. Enter your password when prompted.


## Transferring files

### WinSCP (graphical)

WinSCP provides a drag-and-drop file manager for transferring files to and from the cluster.

1. Download WinSCP from <https://winscp.net>.
2. Create a new connection:
   - **File protocol:** SFTP
   - **Host name:** `fend01.hpc.ku.dk`
   - **Port:** `22`
   - **User name:** your HPC username
3. Click **Login** and enter your password.
4. Drag files between your local machine (left panel) and the cluster (right panel).

### `scp` from the command line

If you prefer the command line, use `scp` from PowerShell or Windows Terminal:

```
# Copy a file to the cluster
scp myfile.txt <your-username>@fend01.hpc.ku.dk:/groups/qist/<your-username>/

# Copy a file from the cluster
scp <your-username>@fend01.hpc.ku.dk:/groups/qist/<your-username>/results.csv .
```

### `scp` from WSL

Works identically to Linux:

```bash
scp myfile.txt <your-username>@fend01.hpc.ku.dk:/groups/qist/<your-username>/
```
