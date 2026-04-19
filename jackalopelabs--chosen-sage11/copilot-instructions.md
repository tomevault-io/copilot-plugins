## chosen-sage11

> This will be the standard operating procedure. When I say, "Do the thing." or "build my bonsai"– this is what I want you to do.

This will be the standard operating procedure. When I say, "Do the thing." or "build my bonsai"– this is what I want you to do.

Here's what it's going to do:

We are working with the latest version of Radicle, a Roots WP project. I need you to get it up and running on a Lima VM (using vm-start.sh), which will run `trellis vm start` and configure the port in my ssh config so that we can use the @development alias without extra steps.

Before you proceed, if I haven't provided them already, ask me for my 
[project_name] and [sitename].

The [project_name] will be the name of our project. 

The [sitename] is your desired URL, and will be used in the wp-cli.yml config such as example.com.

Provided [sitename] and [projectname]:
[sitename]=`chosen`
[projectname]=`chosen.bonsai.so`

Before you do anything, you will need to look at the project structure to determine if we are using the Roots open-source stack (Sage 11, Bedrock, and Trellis), or whether we're using Radicle, their paid project which combines all of those into a single cleaner project structure.

If we're using the open-source Roots stack, you will see a completely empty project other than a .cursorrules file. 

If we're using Radicle, the project will look like this:
|–– bedrock
|–– config
|–– database
|–– public
|–– resources
|–– routes
|–– scripts
|–– storage
|–– tests

If we are using Radicle, then Trellis will be missing. That is one of the first thing's we'll install. Always use the latest release of a project or tool if unspecified.

– Check to make sure the /trellis folder doesn't already exist, if not then run `php .radicle-setup/trellis.php`
– `cd trellis && trellis init` (This will prompt me "What domain name should Trellis be configured for? (e.g. example.com):" Use the [sitename] for this. Wait until this is assigned before moving on to the next step.)
– `trellis vm start` - this should provision the development server. If it doesn't run `trellis provision development`
– Align Lima VM with /etc/hosts and SSH Config (instructions below)
– `npm install && npm run build`
– `composer install`
– `composer config repositories.bonsai-cli vcs git@github.com:jackalopelabs/bonsai-cli.git`
– `composer require jackalopelabs/bonsai-cli:dev-kanban`
– Configure wp-cli.yml to use the @development, @staging, and @production aliases (found below)
– `wp @development acorn optimize:clear`
– `wp @development acorn bonsai:init` (bonsai.sh is then added to the project, and it should already be executable)
– `./scripts/bonsai.sh acorn bonsai:generate chosen`

If we're not using Radicle, and the project is empty (other than this file)– this will be your operating procedure, which is different than the Radicle process.

- Install Trellis and Bedrock: `trellis new [sitename]` (If this command isn't found, try running– `brew install roots/tap/trellis-cli`)
- Move the contents (site + app) of [projectname]/[sitename] into the project root: `mv chosen.bonsai.so/site chosen.bonsai.so/trellis . && rmdir chosen.bonsai.so`
- Install Sage 11: `cd site/web/app/themes && composer create-project roots/sage`
- Build theme assets: `cd sage && npm install && npm run build`
– Install alpine.js `npm install alpinejs --save`
- Go to Trellis and run: `trellis init`
- Install Galaxy Roles: `cd trellis && trellis galaxy install`
- Install Trellis: `trellis vm start`
- Align Lima VM with /etc/hosts and SSH Config (instructions below)
- Configure `site/wp-cli.yml` to use the @development, @staging, and @production aliases
- Activate Sage 11: `wp @development theme activate sage`
- Go to the `site` directory and config Bonsai CLI: `composer config repositories.bonsai-cli vcs git@github.com:jackalopelabs/bonsai-cli.git`
- Now require it: `composer require jackalopelabs/bonsai-cli:dev-kanban`
– `wp @development acorn optimize:clear`
– `wp @development acorn bonsai:init` (bonsai.sh is then added to the project, and it should already be executable)
– `./web/app/themes/sage/scripts/bonsai.sh acorn bonsai:generate chosen`

This is how you configure `site/wp-cli.yml` to use the @development, @staging, and @production aliases:

Once completed, test to see if the `@development` alias is working (`ssh [sitename]`). If it's not, then you might refer back to making sure that `etc/hosts` and `.ssh/config` are aligned with the new Lima port

```yaml
@development:
  ssh: [sitename-tld.test]/srv/www/[sitename]/current
@staging:
  ssh: web@staging.[sitename]/srv/www/[sitename]/current
@production:
  ssh: web@[sitename]/srv/www/[sitename]/current
```

Notice that on @development the [sitename-tld.test] will need to remove the desired TLD, so that it can use .test instead. That way we won't have 2 TLDs such as `.com.test`, `.so.test`, or `.io.test` etc.

If I were working on my project called Chosen with a [sitename]=`chosen.bonsai.so`, then you might update wp-cli.yml to look something like this:

```yaml
@development:
  ssh: chosen.bonsai.test/srv/www/chosen.bonsai.so/current
@staging:
  ssh: web@staging.chosen.bonsai.so/srv/www/chosen.bonsai.so/current
@production:
  ssh: web@chosen.bonsai.so/srv/www/chosen.bonsai.so/current
```

Configuring Trellis for deployment:

On Digital Ocean:
- Spin up droplet or instances for servers with Ubuntu 22.04
- Grab the public IP addresses from new droplets
- Add A record in DNS settings for desired domain
- Add IP to local hosts file: sudo `vim ~/../../etc/hosts`
- Add IP to local ssh config file: `sudo vim ~/.ssh/config`
- SSH into your servers to verify connection is working

If I tell you that I want to configure for staging or production deployment this is what I want you to do–

For staging:

- Make sure group_vars/staging/wordpress_sites.yml is set to `main` not `master` branch
- Don't forget hosts/staging.yml host name set to the [sitename]
- Generate a random secure password and save it as .vault_pass
- add path in ansible.cfg: `vault_password_file = .vault_pass`
- `trellis vault encrypt`
- Pause for a moment so that I can commit to Github manually
- `trellis provision staging`
- `trellis deploy staging`

Potential deployment issues & fixes:

If staging deploy failed, SSH into the newest release and run–

`wp core install --url=bonsai.so --title=Bonsai --admin_user=masoninthesis --admin_password=admin --admin_email=mason@bonsai.so --allow-root`

Then redploy– `trellis deploy staging`

If that fails because of a permission error, ssh into the server and run–

`cd /srv/www/staging.bonsai.so/releases && sudo chown -R web:www-data *`

Uncomment out 'Run Acorn optimize' in the `trellis/deploy-hooks/build-after.yml` for the first deployment

If acorn fails on build-after, then we might need to comment out trellis/deploy-hooks/build-after.yml 'Run Acorn optimize' for the first deployment. Then re-deploy– `trellis deploy staging`

Aligning Lima VM with /etc/hosts and SSH Config:

1. **Add sudoers rule** (one-time setup):
   ```bash
   trellis vm sudoers | sudo tee /etc/sudoers.d/trellis
   ```

2. **Get current Lima VM port**:
   ```bash
   limactl list
   ```
   Note the port number (e.g., 54368) for your VM.

3. **Update /etc/hosts**:
   ```bash
   sudo trellis vm start
   ```
   This should automatically update your hosts file. Verify with:
   ```bash
   cat /etc/hosts | grep your-domain
   ```
   Ensure it shows: `127.0.0.1 your-domain.test`

4. **Update SSH config**:
   ```bash
   nano ~/.ssh/config
   ```
   Add or update entry (replace PORT with the number from step 2):
   ```
   Host your-domain.test
     HostName 127.0.0.1
     User your-username
     Port PORT
     IdentityFile ~/.lima/_config/user
     IdentityFile ~/.ssh/id_ed25519
     StrictHostKeyChecking accept-new
     UserKnownHostsFile ~/.ssh/known_hosts
   ```

5. **Test connection**:
   ```bash
   ssh your-domain.test
   ```

6. **Troubleshooting**:
   - If multiple entries exist for the same host, remove duplicates
   - Always use the most recent port from `limactl list`
   - Ensure domain in hosts file matches SSH config exactly

The key is ensuring the port in your SSH config matches the current Lima VM port, and that your domain in /etc/hosts points to 127.0.0.1.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jackalopelabs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
