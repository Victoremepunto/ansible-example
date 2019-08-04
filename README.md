# Ansible example project - Restart service on variable change

This is an example Ansible project that aims to show how to create a simple playbook that handles a service restart based on a variable change.

A variable named after `my_variable` is used to define a string, that will be stored in a local file on the path specified by the `output_file` variable.

Default value for `output_file` is `/tmp/var_output.ansible`.

On every playbook run, the expected behavior is the following:

- On first run it is considered that the output\_file doesn't exist, and a backup and a rollback doesn't makes sense. In case of failure the recently created output\_file is not preserved.
- The variable contents are written into path specified by `output_file` variable. It is expected the ansible user has write permissions (no elevation requirements have been considered for simplicity).
- An attempt to restart the service is performed only if there are changes from previous versions. This is achieved by registering and checking the state from the previous task ( copy the variable contents into `output\_file` )
- Restart operation requires elevation (become flag set to `True` , defaulting to `root` user). it is recommended to either provide become password with `--ask-become-pass` flag, configure passwordless sudo for ansible user for instance. 
- On subsequent runs ( in case output\_file exists ) a backup file of this file is created (with .bck suffix) for rollback purposes. The reason is the following: As the original file is replaced with the new variable contents, and a restart is attempted, is something fails, the file would be already updated, and subsequent runs will not attempt to retry the restart task.
- The backup file is always removed in order not to leave traces of previous runs.

The structure of the project is the following:

```
├── hosts                    # Dummy hosts file, including only 127.0.0.1 (localhost)
├── main.yml                 # Main playbook file, to import 'common' tasks for `all` hosts from inventory file
├── README.md                # This file
└── roles                    # Roles directory
    └── common               # 'common` roles directory
        ├── handlers
        │   └── main.yml     # Unused `handler` file to demonstrate handler approach for service restart.
        ├── tasks
        │   └── main.yml     # Common tasks main file.
        └── vars
            └── main.yml     # Variables for `common` tasks
``` 

The playbook project uses the structure recommended by [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)

The playbook only defines a `common` role for simplicity, if more tasks are to be added , simply replicate `common` directory structure for other tasks . For example, this restart tasks can be moved inside of a `webservers` role in case such task only applies to web servers.

