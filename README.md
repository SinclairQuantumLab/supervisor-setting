# supervisor-setting

## Overview
This repository provides instructions and templates to set up `supervisor` for managing background processes (daemons) in Linux and Windows environments. 

The main architecture and goals of this setup are:
- **Conda Environment Isolation:** Instead of installing supervisor globally via `apt`, it is installed in an isolated Conda environment to keep Python dependencies safe and manageable.
- **Systemd Daemonization:** The Conda-installed supervisor is registered as a native systemd service. This ensures it starts automatically on boot and runs reliably in the background.
- **Collaboration-Friendly Permissions:** By changing the group ownership of `/etc/supervisor/` to `users`, multiple researchers or developers (e.g., connecting via SSH or VS Code Server) can easily add, edit, or manage their own `.conf` files without requiring root (`sudo`) privileges.
- **Integration to [`multivisor`](https://github.com/SinclairQuantumLab/multivisor)** (optional): a centralized web UI for monitoring `supervisor`s in different computers.

> **Developer's Note:** the above configuration is from the consideration on that `supervisor` is no longer actively maintained and the highest Python version is limited at 3.13 (for `supervisor-4.3.0` as of 2026/03/08; see [Changelog](https://supervisord.org/changes.html) to check the latest update) (cf. backward compatibility holds down to Python 2). Because of it, it is avoided to install `supervisor` via standard package managers (e.g., `apt` in Debian-based Linux) as it used the system Python which version is often of limited control. Therefore installing `supervisor` in a virtual environment with a compatible python version and manually create a system daemon are taken as an alternative approach.

### Relevant materials
- Official documentation: https://supervisord.org/
- Github: https://github.com/Supervisor/supervisor
- PyPI: https://pypi.org/project/supervisor/
- Changelog: https://supervisord.org/changes.html
- Issue tracker: https://github.com/Supervisor/supervisor/issues

---

## Linux

1. Download `/linux/etc/supervisor/` in the repo and copy the `supervisor` folder in the `/etc/` folder in the computer in which you want to install `supervisor`. Make sure the `supervisor` folder contains all of the below:
    - `conf.d` folder
    - `logs` folder
    - `run` folder
    - `supervisord.conf` file
    
<!-- 
    > **Note:** If they don't exist, create them or supervisor will not work. Below is the stdout of `sudo systemctl status --no-pager supervisor` when supervisor was installed as a systemd service daemon via `sudo apt install supervisor` but failed due to missing folders:

    ```text
    ● supervisor.service - Supervisor process control system for UNIX
        Loaded: loaded (/usr/lib/systemd/system/supervisor.service; enabled; preset: enabled)
        Active: activating (auto-restart) (Result: exit-code) since Mon 2025-03-10 15:40:12 MDT; 6s ago
        Docs: http://supervisord.org
        Process: 3156719 ExecStart=/usr/bin/supervisord -n -c /etc/supervisor/supervisord.conf (code=exited, status=2)
    Main PID: 3156719 (code=exited, status=2)
        Tasks: 21 (limit: 38053)
        Memory: 34.5M (peak: 230.7M)
            CPU: 112ms
        CGroup: /system.slice/supervisor.service
                └─3156573 python ./main.py
    ``` 
-->

### 1. Create an isolated environment for supervisor:
Create the folder where you want to install `supervisor` (recommended: `~/Projects/supervisor/`) and run the below commandline in `bash` terminal: 
```bash
$ conda create -n supervisor python=[compatible python version] supervisor
```
> **NOTE**: As of 2026/03/08, `python==3.13` works with `supervisor-4.3.0`.

### 2. Configure Systemd Service
Copy `/linux/systemd/system/supervisor.service` file in this repo into the computer's `/linux/systemd/system/` foler.
> **cf.** By default, the `.service` files for system-wide services are placed in `/lib/systemd/system/`, while the custom services in `/etc/systemd/system/` override the system-wide services.

Open `supervisor.service` file and replace the `<path to conda env>` placeholders with the actual path that can be found from `conda env list`.

### 3. Create a Symlink
Make a symlink for `supervisorctl` in conda to the local `PATH` (so you can run `supervisorctl` globally just by typing it):
```bash
# e.g., for supervisor installed in the conda env at "/home/[username]/miniconda3/envs/myenv/bin/":
$ sudo ln -s /home/[username]/miniconda3/envs/myenv/bin/supervisorctl /usr/local/bin/supervisorctl
```
Allows to call `supervisorctl` by just with the `supervisorctl` command in terminal
```bash
$ sudo supervisorctl
```

### 4. Setup Group Permissions
Change group to `users` and give read, write & file create group permissions to `/etc/supervisor/` and its subfiles/folders. This allows users to edit them without `sudo` (e.g., through SSH or vscode tunnel).
```bash
$ sudo mkdir /etc/supervisor
$ sudo chown -R :users /etc/supervisor
$ sudo chmod -R g+rwx /etc/supervisor
```

Verify the change:
```bash
<host>:/etc$ ls -l /etc | grep supervisor
drwxrwxr--  2 root                 users                 4096 Apr  7 16:57 supervisor
```

### 5. Multivisor Integration (Optional)
Then uncomment the below lines in the configuration file to connect the supervisor to multivisor:
```ini
[rpcinterface:multivisor]
supervisor.rpcinterface_factory = multivisor.rpc:make_rpc_interface
bind=\*:9002
```

### 6. Adding Supervised Programs
Use `/workspaces/supervisor-setting/linux/etc/supervisor/conf.d/template.conf.template` to create `.conf` files in the same folder for programs managed by supervisor.

---

## Windows

Place the `supervisor` folder in `C:\supervisor`