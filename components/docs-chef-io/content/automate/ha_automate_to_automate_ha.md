+++
title = "Automate to Automate HA"

draft = false

gh_repo = "automate"
[menu]
  [menu.automate]
    title = "Automate to Automate HA"
    parent = "automate/deploy_high_availability/migration"
    identifier = "automate/deploy_high_availability/migration/ha_automate_to_automate_ha.md Automate to Automate HA"
    weight = 220
+++
 
## Upgrade with FileSystem Backup Locally

Here we expect both the versions of Standalone Chef Automate and Chef Automate HA are the same. Chef Automate HA is only available in version 4.x.

1. Create Backup of Chef Automate Standalone using the following command:

```bash
chef-automate backup create
chef-automate bootstrap bundle create bootstrap.abb
```

- The first command will create the backup to the `/var/opt/chef-automate/backup` location unless you specify the location in `config.toml` file.
- The second command will create the `bootstrap.abb`.
- Once the backup is completed, save the backup Id. For example: `20210622065515`.

2. Create **Bundle** using the following command:

```bash
tar -cvf backup.tar.gz path/to/backup/<backup_id>/ /path/to/backup/automatebackup-elasticsearch/ /path/to/backup/.tmp/
```

3. Transfer the `tar` bundle to one of the Chef Automate HA Frontend Nodes.

4. Transfer the `bootstrap.abb` file to all the Chef Automate HA FrontEnd Nodes (both Chef Automate and Chef Infra Server).

5. Go the Chef Automate HA, where we copied the `tar` file. Unzip the bundle using:

```bash
tar -xf backup.tar.gz -C /mnt/automate_backups
```

6. Stop all the services at frontend nodes in Automate HA Cluster. Run the following command to all the Automate and Chef Infra Server nodes:

``` bash
sudo chef-automate stop
```

7. Run the following command at Chef-Automate node of Automate HA cluster to get the applied `config`:

```bash
sudo chef-automate config show > current_config.toml 
```

{{< note >}}

In Automate **4.x.y** version onwards, OpenSearch credentials are not stored in the config. Add the OpenSearch password to the generated config above. For example:

```bash
[global.v1.external.opensearch.auth.basic_auth]
  username = "admin"
  password = "admin"
```

{{< /note >}}

8. Restore in Chef-Automate HA using the following command:

```bash
automate_version_number=4.0.91 ## please change this based on the version of Chef Automate running.
     
chef-automate backup restore /mnt/automate_backups/backups/<backup_id>/ --patch-config current_config.toml --airgap-bundle /var/tmp/frontend-${automate_version_number}.aib --skip-preflight
```

9. Unpack the `bootstrap.abb` file on all the Frontend nodes:

Login to Each Frontend Node and then run after copying the `bootstrap.abb` file.

```bash
chef-automate bootstrap bundle unpack bootstrap.abb
```

10. Start the Service in All the Frontend Nodes with the command shown below:

```bash
sudo chef-automate start
```

## Upgrade with FileSystem Backup via Volume Mount

Here we expect both the versions of Standalone Chef Automate and Chef Automate HA are same. Chef Automate HA is only available in version **4.x**.

1. Create *Backup* of Chef Automate Standalone using the following command:

```bash
chef-automate backup create
chef-automate bootstrap bundle create bootstrap.abb
```

- The first command will create the backup at the file mount location mentioned in the `config.toml` file.
- The second command will create the `bootstrap.abb`.
- Once the backup is completed, save the backup Id. For example: `20210622065515`

2. Detach the File system from Standalone Chef-Automate.

3. Attach and Mount the same file system to the Automate-HA all the nodes:

- Make sure that it should have permission for hab user

4. Stop all the services at frontend nodes in Automate HA Cluster. Run the below command to all the Automate and Chef Infra Server nodes

``` bash
sudo chef-automate stop
```

5. Get the Automate HA version number from the location `/var/tmp/` in Automate instance. For example : `frontend-4.x.y.aib`

6. Run the command at Chef-Automate node of Automate HA cluster to get the applied config:

```bash
sudo chef-automate config show > current_config.toml 
```

**Note:** From Automate `4.x.y` version onwards, OpenSearch credentials are not stored in the config. Add the OpenSearch password to the generated config above. For example:

```bash
[global.v1.external.opensearch.auth.basic_auth]
username = "admin"
password = "admin"
```

7. Run the restore command in one of the Chef Automate node in Chef-Automate HA cluster:
    
```bash
chef-automate backup restore /mnt/automate_backups/backups/<backup_id>/ --patch-config current_config.toml --airgap-bundle /var/tmp/frontend-4.x.y.aib --skip-preflight
```

8. Copy the `bootstrap.abb` file to all the Chef Automate HA FrontEnd Nodes (both Chef Automate and Chef Infra Server).

9. Upack the `bootstrap.abb` file on all the Frontend nodes. `ssh` to Each Frontend Node and run the following command:

```bash
chef-automate bootstrap bundle unpack bootstrap.abb
```

10. Start the Service in All the Frontend Nodes with command shown below:

``` bash
sudo chef-automate start
```
