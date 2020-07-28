# Exercises 5: Command-line Environment

## Job control

1.  From what we have seen, we can use some `ps aux | grep` commands to get our jobs' pids and then kill them, but there are better ways to do it. Start a `sleep 10000` job in a terminal, background it with `Ctrl-Z` and continue its execution with `bg`. Now use [`pgrep`](http://man7.org/linux/man-pages/man1/pgrep.1.html) to find its pid and [`pkill`](http://man7.org/linux/man-pages/man1/pgrep.1.html) to kill it without ever typing the pid itself. (Hint: use the `-af` flags).

    Consult `man pgrep` or `man pkill`:

    ```
    -a
    Include process ancestors in the match list.  By default, the current pgrep or pkill process and all of its ancestors are excluded (unless -v is used).

    -f
    Match against full argument lists.  The default is to match against process names.
    ```

    `pgrep -af "sleep 10000"` prints the pid. `pkill -af "sleep 10000"` kills the process without typing the pid.

1.  Say you don't want to start a process until another completes, how you would go about it? In this exercise our limiting process will always be `sleep 60 &`.
    One way to achieve this is to use the [`wait`](http://man7.org/linux/man-pages/man1/wait.1p.html) command. Try launching the sleep command and having an `ls` wait until the background process finishes.

    ```
    sleep 60 &
    pgrep sleep | wait && ls
    ```

        However, this strategy will fail if we start in a different bash session, since `wait` only works for child processes. One feature we did not discuss in the notes is that the `kill` command's exit status will be zero on success and nonzero otherwise. `kill -0` does not send a signal but will give a nonzero exit status if the process does not exist.
        Write a bash function called `pidwait` that takes a pid and waits until the given process completes. You should use `sleep` to avoid wasting CPU unnecessarily.

```
pidwait()
{
   while kill -0 $1 2>/dev/null
   do
   sleep 1
   done
   ls
}
```

    More about [kill -0](https://unix.stackexchange.com/questions/169898/what-does-kill-0-do). `kill -0` basically checks if a given pid can be killed i.e. exists. (Not exactly equivalent, since some pid exists but cannot be killed without root access or such) Define `pidwait` bash function as above.  The function could be run as below.

    ```
    sleep 60 &
    pidwait $(pgrep sleep)
    ```

    The function acts equivalent as the `wait` solution aforementioned. The difference is that this function does not depend on the `wait` command and hence does not depend on sessions, not limited to its own child processes. If the process finishes the `kill -0 $1` will give a nonzero exit status as the process does not exist. Then it exits the sleeping while loop. (Remind that `$1` is a placeholder for the first argument of the function, and `2>/dev/null` just discards the STDERR.)

## Terminal multiplexer

1. Follow this `tmux` [tutorial](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/) and then learn how to do some basic customizations following [these steps](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/).

   Done. (Can check [iTerm2](https://iterm2.com/) as an alternative.)

## Aliases

1. Create an alias `dc` that resolves to `cd` for when you type it wrongly.

   ```
   alias dc="cd"
   ```

1. Run `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10` to get your top 10 most used commands and consider writing shorter aliases for them. Note: this works for Bash; if you're using ZSH, use `history 1` instead of just `history`.
   ```
   alias src="source"
   alias zzz="sleep 60 &"
   alias v="vim"
   ```

## Dotfiles

Let's get you up to speed with dotfiles.

1. Create a folder for your dotfiles and set up version
   control.
   ```
   mkdir ~/dotfiles
   git init ~/dotfiles
   ```
1. Add a configuration for at least one program, e.g. your shell, with some
   customization (to start off, it can be something as simple as customizing your shell prompt by setting `$PS1`).

   I added [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) and [oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh) on `~/.zshrc`.  
   Some other great [resource](https://www.anishathalye.com/2014/08/03/managing-your-dotfiles/) to consult.

1. Set up a method to install your dotfiles quickly (and without manual effort) on a new machine. This can be as simple as a shell script that calls `ln -s` for each file, or you could use a [specialized
   utility](https://dotfiles.github.io/utilities/).

   A shell script installer could be as below.

   ```
   #!/bin/bash

   files="bashrc vimrc vim zshrc oh-my-zsh git"

   for file in $files; do
      ln -s ~/dotfiles/$file ~/.$file
   done
   ```

   I used [dotfiles.](https://github.com/anishathalye/dotbot)

1. Test your installation script on a fresh virtual machine.

   Turn on the VM first and ssh connect on the localhost terminal.

   ```
   ssh ivan@192.168.35.239
   ```

   [Dotfiles](https://github.com/anishathalye/dotbot) works as well in the VM.

1. Migrate all of your current tool configurations to your dotfiles repository.

   ```
   mv ~/.{vimrc,vim,zshrc,gitconfig,oh-my-zsh} ~/dotfiles
   ```

   I used [init-dotfiles](https://github.com/Vaelatern/init-dotfiles) script.

1. Publish your dotfiles on GitHub.

   After running [init-dotfiles](https://github.com/Vaelatern/init-dotfiles), I made a new Github repository and pushed my dotfiles folder.

   ```
   git remote add origin git@github.com:Ivan-Kim/dotfiles.git
   git push -u origin master
   ```

   My [dotfiles](https://github.com/Ivan-Kim/dotfiles).

## Remote Machines

Install a Linux virtual machine (or use an already existing one) for this exercise. If you are not familiar with virtual machines check out [this](https://hibbard.eu/install-ubuntu-virtual-box/) tutorial for installing one.

1. Go to `~/.ssh/` and check if you have a pair of SSH keys there. If not, generate them with `ssh-keygen -o -a 100 -t ed25519`. It is recommended that you use a password and use `ssh-agent` , more info [here](https://www.ssh.com/ssh/agent).

   Skipped: already set ssh keys from configuring Github account.

1. Edit `.ssh/config` to have an entry as follows

   ```bash
   Host vm
      User username_goes_here
      HostName ip_goes_here
      IdentityFile ~/.ssh/id_ed25519
      LocalForward 9999 localhost:8888
   ```

   By default the SSH configuration file may not exist so you may need to create `ssh config` by `touch ~/.ssh/config`. This file must be readable and writable only by the user, and not accessible by others: `chmod 600 ~/.ssh/config`. When copy-pasting make sure to check for IdentityFile whether it should be `~/.ssh/id_e25519` or `~/.ssh/id_rsa` depending on the filename in which the private key is stored. [(Source)](https://linuxize.com/post/using-the-ssh-config-file/)

1. Use `ssh-copy-id vm` to copy your ssh key to the server.  
   `ssh` command might not work in certain newtorks as they are configured to block any `ssh` requests with firewalls. It will print an error message as below, as I encountered in a McDonalds WiFi...just change the network to home network and it should work.
   ```
   /usr/bin/ssh-copy-id: ERROR: ssh: connect to host vm_ip_address port 22: Operation timed out
   ```
   When installing Ubuntu, if Import SSH Identity option was checked during installation (I did through Github), there should already be SSH key on the remote server and it prints `WARNING: All keys were skipped because they already exist on the remote system.`

1) Start a webserver in your VM by executing `python -m http.server 8888`. Access the VM webserver by navigating to `http://localhost:9999` in your machine.

   Might have to run `python3 -m http.server 8888`.  
   From the lecture notes there is a hint: "_For example, if we execute jupyter notebook in the remote server that listens to the port 8888. Thus, to forward that to the local port 9999, we would do ssh -L 9999:localhost:8888 foobar@remote_server and then navigate to locahost:9999 in our local machine._"  
   Run the following command on the localhost (non-VM) terminal. Note `vm` has been predefined in the `.ssh/config` file from exercise 2.

   ```
   ssh -L 9999:localhost:8888 vm
   ```

1) Edit your SSH server config by doing `sudo vim /etc/ssh/sshd_config` and disable password authentication by editing the value of `PasswordAuthentication`. Disable root login by editing the value of `PermitRootLogin`. Restart the `ssh` service with `sudo service sshd restart`. Try sshing in again.

   Change both `PasswordAuthentication` and `PermitRootLogin` value to `no`. For macOS to restart the `ssh` service should type in

   ```
   sudo launchctl stop com.openssh.sshd
   sudo launchctl start com.openssh.sshd
   ```

   And there you have it, password authentication for SSH disabled including root user. Your server will now only accept key based login and the root user can not login with password. Now ssh login is only allowed with SSH keys and not passwords, adding a layer of extra security. Otherwise the SSH connection would be refused as below.

   ```
   ➜   ssh root@ip_address
   ssh: connect to host 192.168.35.249 port 22: Connection refused
   ➜   ssh username@ip_address -o PubkeyAuthentication=no
   ssh: connect to host 192.168.35.249 port 22: Connection refused
   ```

   [(Source)](https://www.cyberciti.biz/faq/how-to-disable-ssh-password-login-on-linux/)

1) (Challenge) Install [`mosh`](https://mosh.org/) in the VM and establish a connection. Then disconnect the network adapter of the server/VM. Can mosh properly recover from it?

   Type `sudo apt-get install mosh` in the VM terminal. Then establish a connection by typing `ssh vm` in the localhost (non-VM) terminal.

   Disconnecting the network adapter of the server/VM would mean the internet connection being disrupted. Turning WiFi off would suffice for the experiment. Although the network coneection was lost, when the network connection is made again (Turning WiFi on again) the ssh connection remains and is not lost. So yes, `mosh` properly recovers from network disruptions.

1) (Challenge) Look into what the `-N` and `-f` flags do in `ssh` and figure out what a command to achieve background port forwarding.

   ```
   -N      Do not execute a remote command.  This is useful for just forwarding ports.

   -f      Requests ssh to go to background just before command execution.  This is useful if ssh is going to ask for passwords or passphrases, but the user wants it in the background.  This implies -n.  The recommended way to start X11 programs at a remote site is with something like ssh -f host xterm.

   If the ExitOnForwardFailure configuration option is set to ``yes'', then a client started with -f will wait for all remote port forwards to be successfully established before placing itself in the background.
   ```

   In essence, this connection will run forever on the background. (Well, it can be [killed](https://unix.stackexchange.com/questions/83806/how-to-kill-ssh-session-that-was-started-with-the-f-option-run-in-background) albeit cumbersome.) Building on the command for Exercise 4, `-fN` tag can be added for example.

   ```
   ssh -fN -L 9999:localhost:8888 vm
   ```

   For the context on how this command might be helpful, consult this [article.](https://mpharrigan.com/2016/05/17/background-ssh.html)
