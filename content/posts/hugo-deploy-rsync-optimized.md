+++
date = "2020-02-11T14:00:00Z"
lastmod = "2020-02-11T14:00:00Z"
author = "jhaurawachsman"
title = "Hugo Deploy with Rsync (Optimized Settings)"
subtitle = "Steps to optimize and improve Rsync for a faster, enhanced, and more informative Hugo deployment."
feature = "hugo-deploy-rsync-optimized.webp"
+++

Hugo is one of the leading static site generators available, and has many different deployment methods. Deploying a Hugo website using Rsync is a favorite because it __incrementally__ sends only the files that have changed, in other words, it's smokin' fast! However, many of the published Hugo Rsync tutorials (even the official Hugo docs version) can be optimized and improved for a faster, enhanced, and more informative deployment. Let's take a look at how to get this done.

## Assumptions

- A web host and server, such as [Digital Ocean](http://bit.ly/3bzw7Rg), AWS, or VPS
- Access to your web host with SSH
- A Hugo static website that is ready to deploy

# Creating a Hugo Deploy Script

Create a file named `deploy` in the __root__ of your Hugo project. We will execute this script to initiate deployment each time the website is updated:

> When typing/copying the terminal commands below, don't include the prompt portion: `~ %`

```shell
# Assuming a working directory named "sites"
~ % cd sites/domain.com
domain.com % touch deploy
```

Make the file executable, so it can be run as a `script`:

```shell
domain.com % chmod +x deploy
```

Open the file, using your favorite editor, and copy/paste the code below:

```shell
#!/bin/zsh
USER=username
HOST=ip_or_hostname
DIR=domain.com/public/
rsync -azvh --exclude='.DS_Store' --delete public/ ${USER}@${HOST}:~/${DIR}

exit 0
```

Finally, set the variable values to match your remote web host configuration:

- `#!/bin/zsh`: If you use a different shell, update the _shebang_ program
- `USER`: Change `username` to the username on the remote server under who's home directory your website lives
- `HOST`: Change `ip_or_hostname` to the IP address or hostname of your web host
- `DIR`: Change `domain.com/public/` to the web server root of your website

## Deploying Your Hugo Website

Whenever you are ready to deploy and incrementally update your website, simply enter this command in your terminal:

```shell
domain.com % ./deploy
```

Rsync will connect to the remote web host, compare what has changed, delete what no longer exists, add what is new, and update what has changed - all in an instant.

## Rsync Settings Explained

The Rsync options used in the script presented above have been optimized for a faster, enhanced, and more informative deployment. Let's take a look at each option and see what they mean. But first, let's look at the basic command parts, using the official [Rsync Documentation](https://download.samba.org/pub/rsync/rsync.html) for reference.

Access via remote shell:

```shell
rsync [OPTION...] SRC... [USER@]HOST:DEST
```

- `rsync`: Call to the program itself
- `[OPTION...]`: Options to set
- `SRC...`: Local source files or directory. In the script above, we are using the Hugo default of `public/`
- `[USER@]HOST:DEST`: Remote web host ssh user, host, and destination directory

## Rsync Options Optimized

Now we come to the most important part, the optimized Rsync options which create a faster, enhanced, and more informative Hugo website deployment process:

```shell
rsync -azvh --exclude='.DS_Store' --delete
```

Here's what each part does:

- `-azvh`:
    - `-a`: A loaded option which expands to `-rlptgoD`. Known as the _Archive_ option, this setting groups together several useful options into a handy single parameter
    - `-z`: Compress file data during the transfer, resulting in smaller overall transfer and time
    - `-v`: Increase verbosity, which displays more information during runtime
    - `-h`: Output numbers in a human-readable format, making it easier to understand transfer statistics and result
- `--exclude='.DS_Store'`: For macOS users, this prevents these annoying files from ending up on your web host
- `--delete`: Delete extraneous files from the destination directory. If you delete an image from your local website, it will also be removed from the remote web host

## Make It Your Own

Rsync has a very robust set of options and can be customized to suit your needs. Consider enhancing the Rsync options further based on your unique setup. For example, Windows users may consider excluding OS specific files such as `.Thumbs.db`.
