Title: Upgrading Juju software
TODO:  Review required (some things: apt-get, no more --upload-tools)

# Upgrading Juju software

A Juju topology can be divided into two main parts:

- Juju client
- Juju model (usually several)

The software in each of the above are installed and upgraded differently.

The software for the client, which is responsible for issuing commands to
manage Juju models, is overseen by the OS package management system.  The
software for the models, which house machines Juju creates, is managed by Juju
itself. This section is primarily devoted to describing the procedure for
upgrading this model software.


## Upgrading the client software

The client software is managed by the OS package management system. With Ubuntu
this is usually APT. To upgrade the client, therefore, it is a matter of:

```bash
sudo apt-get update
sudo apt-get install juju
```

If you have installed Juju via the snap package:

```bash
sudo snap refresh juju
```

For more installation information and what versions are available, see
[the install page](reference-install.html).
 

## Upgrading the model software

The model software consists of *Juju agents*. These are pieces of software that
run on each machine Juju creates, including controllers.

Overview:

- A controller admin can upgrade any model within that controller.
- A model owner can upgrade that model.
- Upgrades must be applied to the controller model first.
- An upgrade is applied to agents running on all machines across a model.
- During the upgrade, an algorithm will select a version to upgrade to if a
  version is not specified.
- Juju machines request the new software version from the controller. If the
  latter's cache cannot satisfy the request the controller will attempt a
  download from the internet.
- Backups are recommended prior to upgrading the server software. See
  [Backup and restore](./controllers-backup.html).

See [Notes on upgrading Juju software](./models-upgrade-notes.html)
for upgrading details, including what to do when the controller lacks internet
access.

## The upgrade-model command

The `juju upgrade-model` command initiates the upgrade. This will cause all Juju
agents to be restarted. Before proceeding, ensure the model is in good working
order (`juju status`).

Examples:

Upgrade the controller model for the current controller (this must be done before 
other models on the controller can be upgraded) with the newest version available:

```bash
juju upgrade-model -m controller
```

Upgrade the current model by allowing the version to be auto-selected:

```bash
juju upgrade-model
```

Upgrade the model by specifying the version:

```bash
juju upgrade-model --agent-version 2.0.3
```

Track the progress with:

```bash
watch -n3 "juju status --format=tabular"
```

For complete syntax, see the
[command reference page](./commands.html#upgrade-model). The `juju help
upgrade-model` command also provides reminders and more examples.

!!! Warning: 
    The `--upload-tools` option should be not be used by the end user.


## Verifying the upgrade

The verification of a successful upgrade is obtained by:

```bash
juju status
```

If this command does not complete properly or if there are errors displayed in
its output then proceed to the next section.


## Troubleshooting the upgrade

An upgrade of server software that does not lead to 100% success will require
troubleshooting. See
[Troubleshooting environment upgrades](./troubleshooting-upgrade.html).
