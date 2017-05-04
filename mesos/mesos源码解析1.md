### mesos安装

在centos6.6版本上安装：

* 跟docker一样，mesos不太支持centos6.6，应该是跟cgroup相关，所以需要下载一个elrepo-release-6-6.el6.elrepo.noarch.rpm。

* 为了安装gcc4.8+版本，需要slc6-devtoolset.repo，直接通过wget这种方式下载下来的内容不太对，不清楚为什么，只能手动下载。

* scl enable devtoolset-2 bash时出现错误：Unable to open /etc/scl/prefixes/devtoolset-2!，这句话是为了编译mesos而临时改变gcc的版本。原因是devtoolset-2-toolchain没有安装好，这个需要CERN GPG key，但是直接通过rpm --import出错，所以手动下载保存在文件，然后再rpm import。
