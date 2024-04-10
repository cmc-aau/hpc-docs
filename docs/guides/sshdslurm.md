# Interactive VS Code session in a SLURM job
When using the [Visual Studio Code IDE](https://code.visualstudio.com/) to work remotely, you would normally install the [Remote - SSH](https://code.visualstudio.com/docs/remote/ssh) on your local computer and connect to the particular machine over an SSH connection to work there instead. This is useful when you for example need to work closer to raw data to avoid moving large data around, or when you simply need more juice for demanding tasks. In order to use VS Code remotely on a SLURM cluster, however, you can't normally connect directly to a compute node, and connecting directly to login nodes to run things there is **strictly forbidden**. This small guide will show you how you can start a SLURM job and connect VS Code directly to that instead.

???+ Important
      Please note that this should be considered an [interactive job](../slurm/request.md#interactive-jobs) and the SLURM job should therefore be submitted to a partition that has [over-provisioning](../slurm/request/#overprovisioning) set up, where CPU's are shared, because you will likely spend time typing instead of actually keeping the allocated CPU's busy at all times, which is a waste of resources.

## How does it work?
When you normally connect to a remote server, you use an SSH client to connect to the SSH daemon `sshd`, which is a service running in the background on the server. You then simply talk to this daemon using the SSH protocol to execute commands on the server. The trick here is then to start a separate `sshd` process that runs within a SLURM job, which you then connect to instead through a bridge connection on one of the login nodes.

## SLURM job script
Log in on one of the login nodes, copy the following SLURM batch script somewhere, and adjust the resource requirements for your session. Submit the job using `sbatch` as usual, and remember to [cancel](../slurm/jobcontrol/#cancel-a-job) it when you are done. It won't stop when you close the VS Code window before it runs out of time.

```
#!/bin/bash
#SBATCH --output=/dev/null
#SBATCH --job-name=sshdbridge
#SBATCH --time=0-3:00:00
#SBATCH --partition=default-op
#SBATCH --cpus-per-task=4
#SBATCH --mem=6G

# exit on first error or unset variable
set -eu

# find open port (open and close a socket, and reuse the port)
PORT=$(python3 -c 'import socket; s=socket.socket(); s.bind(("", 0)); print(s.getsockname()[1]); s.close()')
scontrol update JobId="$SLURM_JOB_ID" Comment="$PORT"

# check whether SSH host key exists (used for sshd connection authentication, NOT for user authentication)
ssh_host_key="ssh_host_ed25519_key"
if [[ ! -s "${HOME}/.ssh/${ssh_host_key}" ]]
then
  mkdir -p "${HOME}/.ssh/"
  ssh-keygen -t ed25519 -N "" -f "${HOME}/.ssh/${ssh_host_key}"
  chmod 600 "${HOME}/.ssh/${ssh_host_key}"
fi

# start sshd server on the available port
echo "Starting sshd on port $PORT"
/usr/sbin/sshd -D -p "${PORT}" -f /dev/null -h "${HOME}/.ssh/ssh_host_ed25519_key"
```

## Adjust local SSH config
On your local computer, you need to set up a bridge connection through a login node by adding a few lines to your SSH config file. You can find it through VS Code under the "Remote Explorer" side menu by clicking the little cog "Open SSH Config File":

![Edit SSH config](../img/sshconfigvscode.png)

Then add the following line somewhere:

```
Host bio-ospikachu02-sshdbridge
    ProxyCommand ssh bio-ospikachu02.srv.aau.dk "nc \$(squeue --me --name=sshdbridge --states=R -h -O NodeList,Comment)"
```

The host `bio-ospikachu02.srv.aau.dk` must already exist in your SSH config file for it to work. You can use the provided [SSH config template](../../access/#ssh-config-file-template) if you don't have any BioCloud hosts there yet. Save the file and you should now see the new bridge host under "Remote Explorer" (you may need to hit refresh first):

![Connect through bridge host](img/sshdbridgeconnect.png)

Click "Connect in New (or current) Window" and you can now start working once it connects! Whatever you do in VS Code will now run inside a SLURM job!

## Notes
 - You cannot connect through a [SSH jump host](../../access/#using-an-ssh-jump-host), so you must use VPN instead if you are not at AAU.

 - If you want to be able to run multiple interactive jobs at once, you must use a different name for the job and create separate entries in the SSH config for each job.