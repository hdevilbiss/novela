+++
authors = []
date = 2021-01-27T05:00:00Z
excerpt = "No more cowboy coding - Deploy your WordPress website from Git to a shared web host using Ruby Gem Capistrano"
hero = "/images/content/deployrb.jpg"
timeToRead = 7
title = "Deploying WordPress on a Shared Web Host with Capistrano"

+++

It is possible to automate deployment of a WordPress website to a remote shared server using the [Ruby Gem Capistrano](https://capistranorb.com/), which will automate SSH tasks for you in only one line of code. Add on Capistrano Composer or Capistrano NPM to add additional package managers steps.

I was inspired by the [tutorial on YouTube](https://www.youtube.com/watch?v=HwJZ5OtLLg8 "Deploying WordPress with Capistrano") by roots.

Please note that this information is provided as-is. It is up to you to use your SSH and database credentials wisely.

Edit 2021: I have added on content to use OpenSSH in PowerShell in Windows 10.

### Necessary Deployment Tasks

* Know the stage of the deployment: development, staging, or production.
* Identify the correct server, user, and port.
* Be able to SSH into the server with the user’s private key (`id_rsa`) and private key passphrase, without prompt.
* Know the value for `:release_path`, `:shared_path`, and `:linked_dirs`, among other Capistrano parameters.
* Know the origin Git repository and be able to deploy from it via SSH, HTTPS, or deploy key, without prompt.
* Run `composer install` (and other package manager tasks).
* Create a symlink for `public_html`, to point to the latest release folder. This is required when on a shared web host, since the host's DocumentRoot cannot be altered by the non-root user.
* Symlink the `htaccess` file between releases to maintain SSL, caching, and/or password-protection settings.
* Auto-symlink shared files and directories, such as the WordPress `uploads` folder, between deployments
* Remove old releases.
* Rollback to a previous release from a current release
* Do deployment or rollback using only one line of code in the command line.

#### Assumptions for Web Host

These tools are required to be used by the remote web host. Linux command line (Capistrano tasks written for Linux)- Web server; e.g, Apache, nginx, LiteSpeed

* Database server, e.g., MySQL or MariaDB (10.2)
* PHP
* Git
* Composer
* WP-CLI

#### Assumptions for Local Dev Machine

* Ruby
* Bundler
* Capistrano (via Gemfile)
* Capistrano
* Composer (via Gemfile)

#### Assumptions for Git Repo

A [bedrock project]([https://roots.io/bedrock/](https://roots.io/bedrock/ "https://roots.io/bedrock/")) which handles a [subdirectory installation]([https://wordpress.org/support/article/giving-wordpress-its-own-directory/](https://wordpress.org/support/article/giving-wordpress-its-own-directory/ "https://wordpress.org/support/article/giving-wordpress-its-own-directory/")) automatically using `Composer`.

#### Official Capistrano Documentation

The official documentation for [Capistrano](https://capistranorb.com/) and the [README from roots](https://github.com/roots/bedrock-capistrano) will certainly be clearer documentation than this here. However, I wanted to touch on the some of the **roadblocks** which I encountered during my first deployment. I really hope that my rephrasing will help someone else along the way.

### Setting up SSH Keys

To avoid needing to enter SSH key passphrases during deployment, let's use an agent to remember it: [Git Bash](https://gitforwindows.org/ "Git Bash") or [Pageant](https://www.chiark.greenend.org.uk/\~sgtatham/putty/latest.html), for examples.

I will quickly go through the steps to adding a private key and its passphrase to the agent. But first, If you have not done so already, generate a key pair; e.g., using PuTTYGen.

Upload the public key on one line to your remote server; double-check with your web host where it needs to go. Most likely, it will need to be in the `~/.ssh/authorized_keys` file.  
Make sure that your public key is only one 1 line of code! For whatever reason, mine kept copying over as 6 to 7 lines.

#### Pageant

* Open Pageant and click "Add Key".
* Navigate to your `.ppk` private key, and enter the private key passphrase when prompted.

Pageant will remember your private key and passphrase for as long as you do not shut down or restart your computer; however, you can add Pageant to the list of Windows startup tasks and add your private key whenever you start your day.

After adding your  when you load your PuTTY SSH, SFTP, or remote SSH MySQL connection, you will not need to enter any passphrases.

#### Git Bash

Open Git Bash. Start the SSH agent:  
`eval "$(ssh-agent -s)"`

List identities (there should be none):  
`ssh-add -l`

Try to add your key to the Git Bash agent:  
`ssh-add path/to/privatekey`

If you used PuTTYGen to make your key-pair, may receive an error, “invalid file format”.

To fix this, convert the `ppk` private key to an OpenSSH format, as suggested by [this answer](https://stackoverflow.com/a/44391850/12621376) from samthecodingman on Stack Overflow.

Open PuTTYgen, load the private key, and enter the private key passphrase. Next, click Conversions > Export OpenSSH Key. Save this file as `id_rsa` without any file extension. Now, save the private key – passphrase identity in the SSH agent.

`ssh-add path/to/id_rsa`

You can check that it was added.

`ssh-add -l`

Please note that you must do this every time you boot up your computer. After restarting your computer, running `ssh-add -l`, will show no identities.

To make the SSH agent start automatically, add [this code from GitHub](https://docs.github.com/en/github/authenticating-to-github/working-with-ssh-key-passphrases#auto-launching-ssh-agent-on-git-for-windows) to the `.bashrc` or `.profile` file located at the Git installation folder; e.g., `D:\Git\etc\bash.bashrc`

I understand that you will still need to add your key every day when you boot up.

#### OpenSSH on Windows 10

Per a [Stack Overflow answer](https://stackoverflow.com/a/40720527/12621376) given by user tamj0rd2, Windows 10 has an SSH agent baked into it.

This is my preferred option because it does not require me to reboot the SSH client and re-enter my private key every time that my computer starts up.

1. Click on the start menu icon and type in `Manage optional features` and search for `OpenSSH Client` just to confirm it's there.
1. Click on the start menu icon and type `Services`; scroll down to `OpenSSH Authentication Agent` to make sure it's not disabled.
1. The commands `ssh-agent` and `ssh-add` should now work within the PowerShell terminal.

### Lesson Learned about temporary folder

When running `deploy:check`, I was having issues with the `.sh` file check in the `/tmp/` folder.

```shell
rubycap aborted!SSHKit::Runner::ExecuteError: Exception while executing as USER@example.com: git exit status: 128git stdout: Nothing writtengit stderr: fatal: cannot exec '/tmp/git-ssh-APPLICATION-STAGE-MYNAME.sh': Permission deniedfatal: cannot exec '/tmp/git-ssh-APPLICATION-STAGE-MYNAME.sh': Permission deniedfatal: unable to fork
```

I found [a thread on GitHub](https://github.com/capistrano/capistrano/issues/687#issuecomment-35419084) in which user bbiglari suggested a permissions issue.

> _the issue might be the /tmp folder in your deployment machine does not have enough permission to run the script, change the folder /tmp folder to something else ..._

With this clue, I specified the Capistrano variable, `:tmp_dir`, which tells Capistrano to use a path to which my limited-scope, shared host user can write.

```ruby
rubyset :tmp_dir, -> {"#{fetch(:deploy_to)}/tmp"}
```

### Setting the public_html symlink

By default on the shared web host, the `public_html` symlink will point to the wrong level in the `current` release; it needs to point to `web`.

It is very **important that you point public_html to current/web** to avoid exposing your `.env` file to the public!

If this deployment were _not_ on a shared web host, this would be as simple as editing the Apache `DocumentRoot` or something similar.

In this case, though, a custom `deploy.rb` task will be used: see [roots/bedrock issue #76](https://github.com/roots/bedrock/issues/76). The task was originally written by GitHub user Pier-Philip and updated by roots dev swalkinshaw. 

However, if you want to have two different `public_html` locations, one for staging, and one for production, you might want to set the `public_html symlink location in `config/staging.rb` and `config/production.rb`, respectively.

Here is a gist which I wrote up, which only shows a portion of a full `deploy.rb` file. Normally, I keep `:deploy_to` and `:public_symlink_location` separate in stage-specific locations.

{{< gist hdevilbiss 3f6b735fd9037a519171ab49126861f4 >}}

The execute statement in `:release_public_html` will remove the `public_html` directory recursively and forcefully. Then, it will create a symbolic link to the most current release public folder, `current/web`.
This custom task gets hooked in after shared symlinks are made.  

### Don't forget about Composer!

Don't forget to uncomment this line in your `Capfile` to make sure that `composer install` runs during your deployment! This is very important for roots/bedrock because WordPress is installed as a Composer dependency.

**Capfile**

```ruby
require 'capistrano/composer'
```

### Installing a Private Composer Repository on a Remote Server

Here was another roadblock that I encountered when trying to use Composer on a remote server to install WordPress plugin from my private Git repository. Composer was unable to clone the remote.

```shell
composer Failed to execute git clone --no-checkout
```

#### Generate a Personal Access Token

This error message was happening, even though my local machine SSH agent was forwarding the SSH credentials for my Git profile to the remote server. I was able to manually clone the repository on both HTTPS and SSH; nonetheless, when it came down to it, Composer didn't know what to do with this information.

On the [Roots discourse discussion](https://discourse.roots.io/t/private-or-commercial-wordpress-plugins-as-composer-dependencies/13247/16?u=shed-00) for the "Private or Commercial WordPress Plugins as Composer Dependencies" blog post, someone linked this section of the [Composer docs for GitHub oAuth](https://getcomposer.org/doc/06-config.md#github-oauth), which ended up being extremely helpful, though it took me a couple days to figure it out properly.

First, I generated a Personal Access Token for my GitHub account. However, at this point, I did not want to include this information in the repository. Finally, I realized that my remote server was already configured with a `~/.composer/auth.json` file, in which it had a section for github-oauth.

So I pasted my personal access token - accidentally into gitlab-oauth, which was annoying to debug - and... Still received the error when deploying. Why?

My GitHub username had changed, so I had to reconfigure it on the remote server in `.gitconfig`. 

Finally, success! Capistrano was able to use Composer to install a private GitHub repository! ... but then I received an e-mail from GitHub entitled, "[GitHub API] Deprecation notice for authentication via URL query parameters".

The solution was to update Composer on the remote server by installing a local copy of Composer and rewriting the "composer" alias in the Bash `.profile`.

#### Summarizing

1. You can configure your remote server with a Personal Access Token to Composer install your private repositories.
1. You can install this Personal Access Token into `composer.json`, but you can add it to an `~/.composer/auth.json` file on your remote server.
1. Your GitHub username should be correctly set on your remote server in `.gitconfig`.
1. When authenticating with URL query parameters, you should be using at least Composer version 1.9.3 on your remote server to avoid a deprecation notice from GitHub.

#### A side note about composer install vs update

During manual deployment, I made this mistake a few times, and so wanted to mention it.  
When running `git clone` or `git pull`, I would accidentally run `composer update` instead of `composer install` to update dependencies.  
The difference between these Composer commands is that `composer update` will reference the `composer.json` file, and update the `composer.lock` file accordingly. This means there will be a merge conflict between your Git repo and your local installation.  
You must use `composer install` instead, which will reference the `composer.lock` file.  
Thankfully, the `capistrano/composer` gem will handle this step automatically.

### Workflow summarized  

1. Setup all necessary variables, Gems, and custom tasks for `deploy.rb` and relevant stages under `config/*.rb`, including but not limited to `:ssh_options`, `:deploy_to`, and `server`.
2. Ensure SSH agent has the key-passphrase setup.
3. Assuming the stage is "staging", run `bundle exec cap staging deploy:check`.
4. Assuming no errors, login to SFTP or similar tool to make sure all the necessary `:linked_files` and `:linked_dirs` are present on the remote server under the shared folder; e.g., `.env`, `web/app/uploads`, `web/.htaccess`, `web/.htpasswd`.
5. Run, `bundle exec cap staging deploy --dry-run`.
6. Assuming no errors, run, `bundle exec cap staging deploy`. 

You may need to wait a few minutes for `composer install` to finish.  
At this point, if everything has gone to plan, it should be as simple as logging into (or installing first-time) your WordPress website.  
`https://example.com/wp/wp-admin`

Thanks for reading!