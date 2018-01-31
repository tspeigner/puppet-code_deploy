[![Puppet Forge](https://img.shields.io/puppetforge/v/vStone/percona.svg)](https://github.com/tspeigner/puppet-code_deploy)

# Puppet Code Deploy

This module adds a Task for running puppet code deploy <environment>.

For Puppet Enterprise users, this means you can allow users or admins to do a Puppet code deployment without giving them SSH access to your Puppet master! The ability to run this task remotely or via the Console is gated and tracked by the [RBAC system](https://puppet.com/docs/pe/2017.3/rbac/managing_access.html) built in to PE.

## Requirements

This module is compatible with Puppet Enterprise and Puppet Bolt.

* To [run tasks with Puppet Enterprise](https://puppet.com/docs/pe/2017.3/orchestrator/running_tasks.html), PE 2017.3 or later must be used.

* To [run tasks with Puppet Bolt](https://puppet.com/docs/bolt/0.x/running_tasks_and_plans_with_bolt.html), Bolt 0.5 or later must be installed on the machine from which you are running task commands. The master receiving the task must have SSH enabled.

## Usage

### Puppet Enterprise Tasks

With Puppet Enterprise 2017.3 or higher, you can run this task [from the console](https://puppet.com/docs/pe/2017.3/orchestrator/running_tasks_in_the_console.html) or the command line.

Here's a command line example where we are purging the `foo`, `bar`, and `baz` nodes from the Puppet master, `master.corp.net`:

```shell
[nate@workstation]$ puppet task run purge_node agent_certnames=foo,bar,baz -n master.corp.net

Starting job ...
New job ID: 24
Nodes: 1

Started on master.corp.net ...
Finished on node master.corp.net
  bar :
    result : Node purged

  baz :
    result : Node purged

  foo :
    result : Node purged

Job completed. 1/1 nodes succeeded.
Duration: 6 sec
```

### Bolt

With [Bolt](https://puppet.com/docs/bolt/0.x/running_tasks_and_plans_with_bolt.html), you can run this task on the command line like so:

```shell
bolt task run purge_node agent_certnames=foo,bar,baz --nodes master.corp.net
```

## Parameters

* `agent_certnames`: A comma-separated list of Puppet agent certificate names.

## Finishing the Job

If you are on Puppet Enterprise 2017.3 or higher and you only have one Puppet master, you're done. There's nothing else you need to do after running this task.

For everyone else, continue reading...

### Puppetserver Reload

On Puppetserver versions [before](https://puppet.com/docs/puppetserver/5.1/release_notes.html#new-feature-automatic-crl-refresh-on-certificate-revocation) [5.1.0](https://tickets.puppetlabs.com/browse/SERVER-1933), the `puppetserver` process needs to be reloaded/restarted to re-read the certificate revocation list (CRL) after purging a node. **If you are at or above this version, you don't need to restart the `puppetserver` process**.

This task does **not** restart `puppetserver` for you. It may in future versions.

### Compile Masters

If you have Puppet Enterprise with a Master-of-Masters (MoM) and Compile Masters, you don't need to restart puppetserver but you do need to trigger a puppet run on the Compile Masters after purging to completely refresh the CRL and prevent that node from checking in again.

This can be done with the Orchestrator via the Console's Jobs page or the command line, like so:

```shell
puppet job run -q 'resources { type = "Class" and title = "Puppet_enterprise::Profile::Master" and !(certname = "FQDN_of_your_MoM") }'
```

