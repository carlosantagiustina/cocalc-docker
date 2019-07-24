# CoCalc Docker image with additional packages and tools for Odycceus Summer School

modified version mainainer: C. R. M. A. Santagiustina 


[![](https://images.microbadger.com/badges/image/sagemathinc/cocalc.svg)](https://microbadger.com/images/sagemathinc/cocalc "Size and number of layers")

**Run CoCalc for free for a small group on your own server!**

This is a free open-source  multiuser CoCalc server that you can _**very easily**_ install on your own computer.


**LICENSE AND SUPPORT:**
  - Much of this code is licensed [under the AGPL](https://en.wikipedia.org/wiki/Affero_General_Public_License). If you would instead like a more company-friendly MIT license instead (or you just want to support CoCalc), please contact [help@cocalc.com](help@cocalc.com), and we will sell you a single-instance 1-year license for $799.  This also includes some support, though with no guarantees (that costs more).
  - Email [the mailing list](https://groups.google.com/forum/?fromgroups#!forum/cocalc) for community support.

**SECURITY STATUS:**
  - This is _**not blatantly insecure**_ from outside attack: the database has a long random password, user accounts are separate, encrypted SSL communication is used by default, etc.
  - That said, **a determined user with an account can easily access or change files of other users in the same container!** Use this for personal use, behind a firewall, or with an account creation token, so that only other people you trust create accounts.  Don't make one of these publicly available with important data in it and no account creation token! See [issue 2031]( https://github.com/sagemathinc/cocalc/issues/2031).  Basically, use this only with people you trust.
  - See the [open docker-related CoCalc issues](https://github.com/sagemathinc/cocalc/issues?q=is%3Aopen+is%3Aissue+label%3AA-docker).

## Instructions

Install Docker on your computer (e.g., `apt-get install docker.io` on Ubuntu).   Make sure you have at least **15GB disk space free**, then type:

    docker build carlosantagiustina/cocalc:latest  https://github.com/carlosantagiustina/cocalc-docker.git
    docker run --name=cocalc -d -v ~/cocalc:/projects -p 443:443 carlosantagiustina/cocalc

wait a few minutes for the image to pull, decompress and the container to start, then visit https://localhost.  It is expected that you'll see a "Your connection is not private" warning, since you haven't set up a security certificate.  Click "Show advanced" and "Proceed to localhost (unsafe)".

NOTES:
 - This Docker image only supports 64-bit Intel.
 - If you get an error about the Docker daemon, instead run `sudo docker ...`.
 - CoCalc will NOT work over insecure port 80.  A previous version of these direction suggested using -p 80:80 above and visiting http://localhost, [which will not work](https://github.com/sagemathinc/cocalc/issues/2000).
 - If you are using Microsoft Windows, instead make a docker volume and use that for storage (I'm not sure how easy it is though then to backup files):
    ```
    docker build carlosantagiustina/cocalc:latest  https://github.com/carlosantagiustina/cocalc-docker.git
    docker volume create cocalc-volume
    docker run --name=cocalc -d -v cocalc-volume:/projects -p 443:443 carlosantagiustina/cocalc
    ```

The above command will first build the image, create an empty volume, then start CoCalc, storing your data in the directory `~/cocalc` on the empty volume just created in your computer. If you want to store your worksheets and edit history elsewhere, change `~/cocalc` to something else.  Once your local CoCalc is running, open your web browser to https://localhost.

The docker container is called `cocalc` and you can refer to the container and use commands like:

```
$ docker stop cocalc
$ docker start cocalc
```

You can watch the logs:

```
$ docker logs cocalc -f
```

However, these logs sometimes don't work.  In that case get a bash shell in the terminal and look at the logs using tail:

```
$ docker exec -it cocalc bash
$ tail -f /var/log/hub.log
```

### Clock skew on OS X

It is **critical** that the Docker container have the correct time, since CoCalc assumes that the server has the correct time.
On a laptop running Docker under OS X, the clock will probably get messed up any time you suspend/resume your laptop.  This workaround might work for you: https://github.com/arunvelsriram/docker-time-sync-agent/.


### SSH port forwarding

If you're running this docker image on a remote server and want to use ssh port forwarding to connect, type:
```
ssh -L 8080:localhost:443 username@remote_server
```
then open your web browser to https://localhost:8080

For **enhanced security**, make the container only listen on localhost:
```
docker stop cocalc
docker rm cocalc
docker run --name=cocalc -d -v ~/cocalc:/projects -p  127.0.0.1:443:443 carlosantagiustina/cocalc
```

Then the **only way** to access your CoCalc server is to type the following on your local computer:

    ssh -L 8080:localhost:443 username@remote_server

and open your web browser to https://localhost:8080

### SSH into a project

Instead of doing:
```
docker run --name=cocalc -d -v ~/cocalc:/projects -p 443:443 carlosantagiustina/cocalc
```

do this:

```
docker run --name=cocalc -d -v ~/cocalc:/projects -p 443:443 -p <your ip address>:2222:22  carlosantagiustina/cocalc
```

Then you can do:
```
ssh projectid@<your ip address> -p 2222
```

Note that `project_id` is the hex id string for the project *without hyphens*. One way to show project id in this format is to open a .term file in the project and run this command: (This only works in CoCalc in Docker; USER is set differently in production CoCalc.)

```
echo $USER
```

To use SSH key authentication with the Docker container, have your private key file in the usual place in the host computer, for example `~/.ssh/.id_cocalc`, and copy the matching public key into your project's home directory. For example, you could do the following in a .term in your project:
```
cd
mkdir .ssh
chmod 700 .ssh
vi .ssh/authorized_keys
... paste in contents of ~/.ssh/id_cocalc.pub from host computer ...
chmod 600 .ssh/authorized_keys
```


### Make a user an admin

Get a bash shell insider the container, then connect to the database and make a user (me!) an admin as follows:
```
$ docker exec -it cocalc bash
root@931045eda11f:/# make-user-admin carlosantagiustina@gmail.com
```
Obviously, you should really make the user you created (with its email address) an admin, not me!
Refresh your browser, and then you should see an extra admin panel in the lower right of accounts settings; you can also open any project by directly visiting its URL.

### Change a user's password

It's imposible to recover a password, since only a hash of the password is stored. However, you can change any user's password.

Basically, do what it says in the previous section, but use `reset_password` instead of `make_user_admin`:
```
$ docker exec -it cocalc bash
root@931045eda11f:/# coffee
coffee> require 'c'
coffee> db.reset_password(email_address:'a@b.c', cb:done())
undefined
coffee> Password changed for a@b.c
Random Password:

                146dba6b28ba96ee555c7144a43c21f2
```

That will change the person's password to the big random string; you can then email it to them.

#### Account Creation Token

After making your main account an admin as above, search for "Account Creation Token" in the Admin tab. Put some random string there and other people will not be able to create accounts in your CoCalc container, without knowing that token.

### Terminal Height

If `docker exec -it cocalc bash` doesn't seem to give you the right terminal height, e.g. content is only displayed in the uppper part of the terminal, this workaround may help when launching bash:

```
docker exec -e COLUMNS="`tput cols`" -e LINES="`tput lines`" -it cocalc bash
```

More information on this issue is in [moby issue 33794](https://github.com/moby/moby/issues/33794).


### Installation for SELinux (Fedora, etc.)

In order to build and run CoCalc on an SELinux box, first set SELinux to permissive:

```
$ setenforce 0
```

<Install cocalc>

Tell docker and SELinux to "play nicely":

```
$ chcon -Rt svirt_sandbox_file_t cocalc
```

return SELinux to enabled:
```
$ setenforce 1
```

-- via [discussion](https://groups.google.com/forum/#!msg/cocalc/nhtbraq1_X4/QTlBy3opBAAJ)


## Your data

If you started the container as above, there will be a directory ~/cocalc on your host computer that contains **all** data and files related to your projects and users -- go ahead and verify that it is there before upgrading. It might look like this:
```
carlosantagiustina:~ carlosantagiustina$ ls cocalc
be889c14-dc96-4538-989b-4117ffe84148	postgres    conf
```

The directory `postgres` contains the database files, so all projects, users, file editing history, etc. The directory conf contains some secrets and log files. There will also be one directory (like `be889c14-dc96-4538-989b-4117ffe84148`) for each project that is created.


