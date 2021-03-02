# mycli通过ssh连接MySQL服务问题

### What's mycli

>  a command line client for MySQL that can do auto-completion and syntax highlighting.

官网地址: https://www.mycli.net/

Github: https://github.com/dbcli/mycli

### Usage:

mycli --help

```
Usage: mycli [OPTIONS] [DATABASE]

  A MySQL terminal client with auto-completion and syntax highlighting.

  Examples:
    - mycli my_database
    - mycli -u my_user -h my_host.com my_database
    - mycli mysql://my_user@my_host.com:3306/my_database

Options:
  -h, --host TEXT               Host address of the database.
  -P, --port INTEGER            Port number to use for connection. Honors
                                $MYSQL_TCP_PORT.

  -u, --user TEXT               User name to connect to the database.
  -S, --socket TEXT             The socket file to use for connection.
  -p, --password TEXT           Password to connect to the database.
  --pass TEXT                   Password to connect to the database.
  --ssh-user TEXT               User name to connect to ssh server.
  --ssh-host TEXT               Host name to connect to ssh server.
  --ssh-port INTEGER            Port to connect to ssh server.
  --ssh-password TEXT           Password to connect to ssh server.
  --ssh-key-filename TEXT       Private key filename (identify file) for the
                                ssh connection.

  --ssh-config-path TEXT        Path to ssh configuration.
  --ssh-config-host TEXT        Host to connect to ssh server reading from ssh
                                configuration.

  --ssl-ca PATH                 CA file in PEM format.
  --ssl-capath TEXT             CA directory.
  --ssl-cert PATH               X509 cert in PEM format.
  --ssl-key PATH                X509 key in PEM format.
  --ssl-cipher TEXT             SSL cipher to use.
  --ssl-verify-server-cert      Verify server's "Common Name" in its cert
                                against hostname used when connecting. This
                                option is disabled by default.

  -V, --version                 Output mycli's version.
  -v, --verbose                 Verbose output.
  -D, --database TEXT           Database to use.
  -d, --dsn TEXT                Use DSN configured into the [alias_dsn]
                                section of myclirc file.

  --list-dsn                    list of DSN configured into the [alias_dsn]
                                section of myclirc file.

  --list-ssh-config             list ssh configurations in the ssh config
                                (requires paramiko).

  -R, --prompt TEXT             Prompt format (Default: "\t \u@\h:\d> ").
  -l, --logfile FILENAME        Log every query and its results to a file.
  --defaults-group-suffix TEXT  Read MySQL config groups with the specified
                                suffix.

  --defaults-file PATH          Only read MySQL options from the given file.
  --myclirc PATH                Location of myclirc file.
  --auto-vertical-output        Automatically switch to vertical output mode
                                if the result is wider than the terminal
                                width.

  -t, --table                   Display batch output in table format.
  --csv                         Display batch output in CSV format.
  --warn / --no-warn            Warn before running a destructive query.
  --local-infile BOOLEAN        Enable/disable LOAD DATA LOCAL INFILE.
  --login-path TEXT             Read this path from the login file.
  -e, --execute TEXT            Execute command and quit.
  --init-command TEXT           SQL statement to execute after connecting.
  --charset TEXT                Character set for MySQL session.
  --help                        Show this message and exit.

```



由于公司的MySQL服务是部署在内网的，必须通过一个代理机器才能访问到

#### 方法一

通过拨VPN之后能访问到，因为连接vpn ip就变成了VPN所在服务器的IP， 这个机器是能连接到MySQL的服务的。因为拨VPN跟科学上网冲突，并且会拖慢网速，因为VPN机器的带宽有限。



#### 方法二 （主要讨论这个方法)

mycli 是支持走 ssh 代理的，所以可以在连接MySQL时指定对应的 SSH服务信息

```sh
mycli -u u_super_hero -h 172.16.32.3 -p {passwd} --ssh-user root --ssh-host  {ssh ip} --ssh-port 22 --ssh-key-filename {ssh key file}
```

这里必须一个插件的支持  `paramiko` 会让你安装 `pip install paramiko`



### 坑来了

再次执行 mycli 连接时，发现还提示需要安装 `paramiko` 但是我明明安装成功了。



#### 猜测1

是否`python`版本有问题，因为当时我使用的环境是 `conda`，我尝试关闭了 conda环境，但还是提示需要安装 `paramiko`



#### 猜测2

此时我使用 `python` 命令，打印的版本是 `python2.7`

但是 `pip` 用的 `python`版本 却是 `python3`，我强烈怀疑是不是因为这个地方有问题

我的第一想法就是把 `ptyhon2.7`给删了，但是发现删除不了，到网上一搜 发现不能轻易删除 自带的 python2.7

因为 macOS 自带的一些软件会依赖 `python2.7` 

于是我又萌生了另一个想法，用 `python3`代替 `python2.7`

我尝试在 `~/.zshrc`中添加以下命令

```shell
alias python=/usr/local/Cellar/python@3.9/3.9.2_1/bin/python3.9
```

保存后使用 `source ~/.zshrc`使其生效

再运行 `python`时 此时已经变成了 `python3`了

我怀着期待的心情再尝试运行一遍 `mycli`，那个让我恐惧的 `paramiko` 提示又出现了



#### 猜测4

我努力平静了好一会，又贼心不死的想要继续解决这个问题

因为我一直怀疑是我本地环境的问题 (在网上找这个问题没有找到想要的答案，故此猜测)

于是我尝试在云服务器的尝试一波，验证下我的想法

尝试的环境如下: `centos7 ,  python3`

```shell
pip install mycli  # 因为用 yum install mycli 找不到包
pip install paramiko # 安装这个可怕的依赖
```

抱着尝试的心态运行

```shell
mycli -u u_super_hero -h 172.16.32.3 -p {passwd} --ssh-user root --ssh-host  {ssh ip} --ssh-port 22 --ssh-key-filename {ssh key file}
```

**居然成功了 ! ! !**

那环境问题就石锤了。



#### 解决

其实解决也是误打误撞，中间尝试了很多次，这里就不赘述了，直接上解决方案

其实答案就在 **猜测4**中，在 `centos7` 我安装 `mycli`时，是使用 `pip`安装的，在 `macOS`上我使用的是 `brew install mycli`这就是唯一的区别。



### 总结

虽然解决了问题，但也是误打误撞

但是也学习到了一些东西， 那就是多尝试 :smiley:

