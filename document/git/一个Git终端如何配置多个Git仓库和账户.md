通常情况下，很多公司会将代码托管到GitLab或者第三方平台上（如阿里）进行管理，而我们自己的开源项目等通常是托管到GitHub上，每个托管网站都对应一个Git账户。
默认情况下，一台电脑上的Git对应一个Git账户，也就只能往一个网站push代码，很不方便，尤其是对于用自己的电脑用来办公，操作不好，很容易产生冲突。

本篇文章将以双账户为例，详细介绍如何在一个Mac Git终端配置多个账户，同时管理多个托管网站的代码。

1.为每个Git账户各生成一对密钥

```
1）进入到 .ssh目录下，利用账户邮箱生成密钥 
cd ~/.ssh
ssh-keygen -t rsa -C "邮箱名"

2）执行上述命令后，会有如下提示
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/bigdata/.ssh/id_rsa):

冒号后是给生成的密钥起个名字，每个账户要区分开来，以github为例，命名为id_rsa_github

3）接下来的提示都直接进行回车，直到秘钥生成

```
生成的私钥名默认是id_rsa，需要为每个私钥单独命名，上述是针对GitHub上的账户处理。

对于自己公司的代码管理网站以GitLab为例，可以用自己的公司邮箱进行上述步骤的处理，私钥采用默认也行，只要与其他账户区分开来即可。

2.将私钥添加到本地

```
ssh-add ~/.ssh/id_rsa
ssh-add ~/.ssh/id_rsa_github
```
然后通过ssh-add -l 查看是否添加成功。如果不明白ssh-add代表什么意思，可执行ssh-add --help查看帮助。

3.对本地私钥进行配置，生产config文件

同样是 .ssh目录下，创建一个config文件，内容如下：

```
#网站的别名
Host gitlab.xxx.com
# 托管网站的域名
HostName gitlab.xxx.com
#指定优先使用哪种方式验证, 支持密码和秘钥验证方式
PreferredAuthentications publickey
# 托管网站上的用户名, 最好写账户邮箱, 否则容易设置失败
User gitlab@126.com
# 使用的密钥文件
IdentityFile ~/.ssh/id_rsa

# 其他
Host github.com
HostName github.com
PreferredAuthentications publickey
User github@126.com
IdentityFile ~/.ssh/id_rsa_github
```

4.将公钥添加到托管网站

将.ssh目录下的id_rsa.pub和id_rsa_github.pub中的内容复制，分别设置到GitLab和GitHub的SSH keys中

5.测试是否配置成功

使用config文件中配置的别名，对于GitLab和GitHub，分别执行：

```
ssh -T gitlab.com
ssh -T github.com
```
提示类似如下信息，就说明配置成功了：

```
Hi username! You’ve successfully authenticated, but GitHub does not provide shell access.
```

执行完上述步骤后，可以通过项目的Git地址，进行clone、pull、commit、push等的测试，来进一步验证。

但是因为存在双/多账户，需要通过Git配置不同账户的用户和邮箱，这里采取GitLab用全局用户名和邮箱配置，而GitHub针对每个clone到本地的项目进行单独的用户名和邮箱配置，避免冲突。

1.配置全局的用户名和邮箱

```
// 将GitLab设置为全局
git config --global user.name "gitlab"
git config --global user.email "gitlab@126.com"
```

2.为GitHub上的每个repository/项目单独设置用户名和邮箱

```
// 注意：下述命令要到每个本地GitHub项目目录下执行
git config user.name "github"
git config user.email "github@126.com"
```

3.最后进行可以代码的修改，拉取、提交、push等，验证配置是否成功

上述都配置、测试成功后，我们就可以在公司项目和自己参与的开源项目之间进行不同的管理和使用了。