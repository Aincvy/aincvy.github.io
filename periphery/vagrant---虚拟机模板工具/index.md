# Vagrant   虚拟机模板工具


# 简介

如果需要频繁的创建和销毁虚拟机，这个工具还是有作用的，否则还是手动执行比较方便。   
Vagrant 需要一个虚拟机工具来创建虚拟机， 比如 VirtualBox, VMware Fusion, 或者 Hyper-V.

预期的工作流程
- 创建 `Vagrantfile`,  编辑一下
- `vagrant up`  自动创建并部署一个虚拟机
- `vagrant ssh`   打开 ssh 会话， 开始做其他事情。

大致上也是这么个流程， 就是可能会发生不少预期之外的情况，不过， 当笔者做了自己的 box之后， 意外情况就少了很多。

*笔者使用 VirtualBox 作为示例后端。*

# 主要内容

## 创建第一个虚拟机

第一步， 寻找镜像。 当然自己做也是可以的。  
打开网址 https://app.vagrantup.com/boxes/search  即可搜索镜像。

示例 镜像： https://app.vagrantup.com/debian/boxes/bullseye64  
可以看到 有一段话是 `How to use this box with Vagrant:`  ，下面给了一个 `VagrantFile` 一个 `New` 的选项。   
这里使用 `New` 的选项所提示的命令。
```shell
cd ~
mkdir debian-bullseye && cd debian-bullseye
vagrant init debian/bullseye64
# 修改 VagrantFile  以进行定制化
# ...
# 启动虚拟机
vagrant up
# 开启 ssh 会话
vagrant ssh
```

镜像文件需要从 Vagrant cloud 上下载，基础镜像基本上只包含了一个最小可启动的系统， 大小好像不到1GB。

使用 `vagrant init` 会在当前目录创建一个 叫做`Vagrantfile` 的文件， 这个就是虚拟机的定制文件。  
先来看一下 上面命令创建的模板文件是什么样子的。
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "debian/bullseye64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Disable the default share of the current code directory. Doing this
  # provides improved isolation between the vagrant box and your host
  # by making sure your Vagrantfile isn't accessable to the vagrant box.
  # If you use this you may want to enable additional shared subfolders as
  # shown above.
  # config.vm.synced_folder ".", "/vagrant", disabled: true

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
```

可以看到， 除了注释， 就只有一句 `config.vm.box = "debian/bullseye64"`  这个 `debian/bullseye64` 就是基础镜像， 可以是远端的， 也可以是本地的。 

简单来说， 就是克隆了一个虚拟机， 然后修改了几个配置项。   
后面的使用基本上也是自己做box, 然后利用box 快速创建虚拟机。  *box就是指vagrant的镜像*

使用 `vagrant up` 之后就可以创建一个虚拟机了。 创建好的虚拟机和 vagrant 几乎就没有关系了。  
我们可以在 virtualbox 的管理界面上看到创建好的虚拟机，并且可以进行一些修改。 

如果修改了网络设置， `vagrant ssh` 命令可能就无法使用了。 不过 `ssh user@host` 的方式还是OK的。 

此外， 如果ssh的时候要求你输入用户名密码， 可以尝试 `vagrant`, `vagrant` .
虽然这是一个普通用户， 但是可以无密码使用 `sudo`

## 定制虚拟机

### 网络部分

```ruby
# 端口转发，  guest： 客户机，  host: 宿主机
# config.vm.network "forwarded_port", guest: 80, host: 8080

# 端口转发， 宿主的端口绑定到localhost
# config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

# host-only 网络模式， 需要先在 vbox 里面创建一个 host-only的 net， 然后才能使用。
# config.vm.network "private_network", ip: "192.168.33.10"

# 桥接模式
# config.vm.network "public_network"
```

如果已经创建了虚拟机， 则无法通过这种方式添加新的端口转发， 可以使用 virtualbox 的管理界面或者命令行添加。  

还可以使用 ssh 的方式添加端口转发， 比如：  `vagrant ssh -- -L 192.168.200.101:8001:localhost:8001`  
命令解释： `ssh -L [本地ip:]本地端口:目标地址:目标端口 [USER@]SSH_SERVER`  
本地指发起 ssh 会话的机器。 

{{< admonition "note" >}}
如果设置为 host-only 模式， 但是 ip 没有生效， 可能需要在**宿主机**安装一下 `net-tools` 工具。 
可见：  https://github.com/hashicorp/vagrant/issues/8686
{{< /admonition>}}

### virtual box 虚拟机设置 
```ruby
config.vm.provider "virtualbox" do |vb|
    # 开启机器的时候是否显示 GUI 界面？
    vb.gui = true

    # 虚拟机的名字
    vb.name = "cpp-dev"

    # 内存大小 MB
    vb.memory = "1024"
    # 虚拟机cpu 核心（线程）数量 
    vb.cpus= 4
end
```

可以设置的值和在virtualbox 的管理界面上是一致的。 

### 磁盘部分
```ruby
config.disksize.size = '50GB'
```
需要用到一个虚拟机插件 
```shell
vagrant plugin install vagrant-disksize
```

默认是20GB的空间大小， 在修改了容量之后， vagrant 会自动修改分区大小， 但是需要自己手动同步到 文件系统
```shell
vagrant ssh
sudo resize2fs /dev/sda1
```

### 用户名密码
```ruby
# 在出现 ssh 密钥错误的时候可以尝试使用
# config.ssh.insert_key = false
config.ssh.username = 'root'
config.ssh.password = 'admin123'
```

{{< admonition "warning" >}}
！注意， 如果不是自定义的镜像，用户名不要设置成 `root` ! 因为 sshd 默认的配置不允许root 登录， 也不允许密码登录。 
{{< /admonition>}}

因为普通用户可以无密码`sudo` 所以也没必要非得直接创建 root 用户。  
如果读者说 诶，我就是想要 💓, 那么你可以参考这个地址： https://stackoverflow.com/questions/25758737/vagrant-login-as-root-by-default

大致操作步骤就是先使用一个普通用户 登录进去， 然后修改sshd_config 文件，允许root登录， 允许密码登录。   
*有时候 vagrant 在插入密钥的时候会失败，这个时候就需要允许密码登录了。*   
并且创建目录 `/root/.ssh`    
然后关闭虚拟机， 打包成一个自定义的box。


### 自定义box
box 呢， 就是虚拟机模板， 我们可以利用现有虚拟机来创建一个box 。 以供我们下次使用。 

下面的命令示意如何创建一个自定义box。 
```shell
$ vboxmanage list vms
"<inaccessible>" {d17e3814-8a95-4d83-a7fc-13e6e3cdb10c}
"ubuntu-22-lts-copy" {d4c01226-bdac-40c4-bb8a-19d711d9596c}
"omv" {0536c357-c116-4763-b57c-495c5e9afbfa}
"k8s-minikube" {74006850-c9d4-4fe2-9341-ae6409eb26bd}
"cpp-dev" {6e432b59-f15a-404f-9f69-bb7acfa4e84d}

$ vagrant package --base "cpp-dev"
...
...
==> cpp-dev: Compressing package to: /mnt/ssd1/default/cpp-dev/package.box

$ mkdir -p ~/vagrant-box/cpp-dev
$ mv package.box ~/vagrant-box/cpp-dev
$ vagrant box add --name="cpp-dev" ~/vagrant-box/cpp-dev/package.box
...
...
==> box: Successfully added box 'cpp-dev' (v0) for 'virtualbox'!

$ vagrant box list
cpp-dev             (virtualbox, 0)
debian/bullseye64   (virtualbox, 11.20230602.1)

```

至此， 我们就添加了一个自定义的box。 那么如何使用呢？ 
很简单， 只需要这样就可以了。   
`config.vm.box = "debian/bullseye64"  =>   config.vm.box = "cpp-dev"`  
把值修改成  `vagrant box add --name="cpp-dev"` 中， --name 定义的值就可以了。 

当从 vagrant cloud 上新下载一个镜像之后， 你可能需要手动的安装 virtualbox guest additions 。  
安装完成之后， 就可以制作第一个自己的box 了。  
*open-vm-tools 好像和virtualbox是不通用的。。 必须安装virtualbox guest additions。*

### 虚拟机磁盘文件位置
默认情况下， 创建的虚拟机放在了 virtualbox 的默认虚拟机文件夹里面。
想要修改的话， 可以创建一个shell 脚本， 内容大致如下： 
```shell
VBoxManage setproperty machinefolder /custom/path
vagrant up
VBoxManage setproperty machinefolder default
```

### 销毁虚拟机
进入VagrantFile 所在的目录， 执行命令 `vagrant destroy ` 即可。

# 其他

扩展阅读
- [vagrant-how-to-specify-the-disk-size](https://stackoverflow.com/a/51064467/11226492)
- https://stackoverflow.com/questions/25758737/vagrant-login-as-root-by-default
- https://linuxize.com/post/how-to-setup-ssh-tunneling/
- https://github.com/hashicorp/vagrant/issues/8686
- https://www.virtualbox.org/manual/UserManual.html#vboxmanage


---

> 作者: Aincvy  
> URL: https://fantasyplayer.link/periphery/vagrant---%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%A8%A1%E6%9D%BF%E5%B7%A5%E5%85%B7/  

