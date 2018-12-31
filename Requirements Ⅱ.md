## Requirements Ⅱ

搭建环境:

* Tencent clout dual-core 4GB server
* Ubuntu 16.04
* Docker 17.03

我们选取了 RKE 来搭建 Kubernetes 环境，因为开始没注意到 RKE 的一些特殊要求所以耗费了较多的时间，但是如果正常搭建是很快的。下面是搭建步骤以及注意事项：

1. 下载安装对应版本的 RKE

    在 [rke-release](https://github.com/rancher/rke/releases/) 下载对应版本的 rke 到 server 上

   `wget rke-release-url` 

   `mv rke-download rke`

   `chmod +x ./rke`  *将 rke 更改成可执行文件*

2. 安装对应版本的 Docker

   这一步需要比较注意，因为 rke 只支持到17.03版本的 docker，但是现在最新版的 docker 是18.09，所以我们需要安装对应版本的 docker. 最开始没有注意导致失败，后来在网上找到的方法步骤繁琐且非常的不好用，最后发现 rke 的 requirement 里就附上了安装方法。算是个教训了，动手之前还是应该把要求好好看完，而不是贸然动手，磨刀不误砍柴工。

   | Docker Version | Install Script                                               |
   | -------------- | ------------------------------------------------------------ |
   | 17.03.2        | `curl https://releases.rancher.com/install-docker/17.03.sh | sh` |
   | 1.13.1         | `curl https://releases.rancher.com/install-docker/1.13.sh | sh` |
   | 1.12.6         | `curl https://releases.rancher.com/install-docker/1.12.sh | sh` |

3. 配置 ssh

   RKE的工作方式是通过SSH连接到每个服务器，并在此服务器上建立到Docker socket的隧道，这意味着SSH用户必须能够访问此服务器上的Docker引擎。

   所以我们建立单节点的 kubernetes 的时候，也要确保自己能 ssh 到本机。

   在 `.ssh` 目录下用 `ssh-keygen` 生成一堆秘钥

   然后 `cat public_key >> authorized_keys` 将生成的秘钥的公钥添加到 authorized_keys里

   之后可以 ssh server_ip ，此时应该可以 ssh 上服务器本机了。

4. 配置 rke config

   rke 的 cluster.yml 可以通过 ./`rke config --name cluster.yml` 来进行交互式的创建

5. 运行 rke

   创建了对应的 cluster.yml 就可以通过

   `./rke up` 命令来创建 kubernetes 集群了

   如果最后一行是 `Finished building Kubernetes cluster successfully` 代表集群创建成功。

6. 安装 kubectl

   接下来需要安装 kubectl 工具来进行集群的管理，由于官方的安装方法被墙了，推荐到 GitHub 上获取对应 release 的下载地址后用命令行下载手动安装。

   ![image-20181231142258289](https://ws3.sinaimg.cn/large/006tNbRwly1fypwq0rsiij30y10n078g.jpg)

   手动安装方法：

     1. 到 [kubectl release](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG.md#client-binaries-1) ，点击需要的版本
         ![image-20181231142901834](https://ws3.sinaimg.cn/large/006tNbRwly1fypww4ryrjj30ra0hbtaa.jpg)

     2. 找到对应的 Client Binaries，获取下载地址

         ![image-20181231143139588](https://ws1.sinaimg.cn/large/006tNbRwly1fypwywg9owj30qs0rrals.jpg)

     3. 下载kubectl包，解压后，将kubectl命令赋予权限和拷贝到用户命令目录下

         ```shell
         wget download-link
         tar -zxvf kubernetes-client-linux-amd64.tar.gz
         cd kubernetes/client/bin
         chmod +x ./kubectl
         sudo mv ./kubectl /usr/local/bin/kubectl
         ```

     4. 运行 `kubectl version` ，返回版本信息，说明安装成功

         ```shell
         kubectl --kubeconfig kube_config_cluster.yml version
         
         Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.0", GitCommit:"fc32d2f3698e36b93322a3465f63a14e9f0eaead", GitTreeState:"clean", BuildDate:"2018-03-27T00:13:02Z", GoVersion:"go1.9.4", Compiler:"gc", Platform:"darwin/amd64"}
         Server Version: version.Info{Major:"1", Minor:"8+", GitVersion:"v1.8.9-rancher1", GitCommit:"68595e18f25e24125244e9966b1e5468a98c1cd4", GitTreeState:"clean", BuildDate:"2018-03-13T04:37:53Z", GoVersion:"go1.8.3", Compiler:"gc", Platform:"linux/amd64"}
         
         kubectl --kubeconfig kube_config_cluster.yml get nodes
         NAME            STATUS    ROLES                      AGE       VERSION
         10.0.0.1         Ready     controlplane,etcd,worker   35m       v1.10.3-rancher1
         ```

   至此我们完成了搭建工作，可以用 kubectl 控制我们的集群了。接下来就可以开始部署工作了。



