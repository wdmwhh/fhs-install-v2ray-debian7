# fhs-install-v2ray

> 欲查阅以简体中文撰写的介绍，请访问：[README.zh-Hans-CN.md](README.zh-Hans-CN.md)

> Bash script for installing V2Ray in operating systems such as Debian / CentOS / Fedora / openSUSE that support systemd

該腳本安裝的文件符合 [Filesystem Hierarchy Standard (FHS)](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)：

```
installed: /usr/local/bin/v2ray
installed: /usr/local/bin/v2ctl
installed: /usr/local/share/v2ray/geoip.dat
installed: /usr/local/share/v2ray/geosite.dat
installed: /usr/local/etc/v2ray/config.json
installed: /var/log/v2ray/
installed: /var/log/v2ray/access.log
installed: /var/log/v2ray/error.log
installed: /etc/systemd/system/v2ray.service
installed: /etc/systemd/system/v2ray@.service
```

## 重要提示

**不推薦在 docker 中使用本專案安裝 v2ray，請直接使用 [官方映象](https://github.com/v2fly/docker)。**  
如果官方映象不能滿足您自定義安裝的需要，請以**復刻並修改上游 dockerfile 的方式來實現**。  

本專案**不會為您自動生成配置檔案**；**只解決使用者安裝階段遇到的問題**。其他問題在這裡是無法得到幫助的。  
請在安裝完成後參閱 [文件](https://www.v2fly.org/) 瞭解配置檔案語法，並自己完成適合自己的配置檔案。過程中可參閱社群貢獻的 [配置檔案模板](https://github.com/v2fly/v2ray-examples)  
（**提請您注意這些模板複製下來以後是需要您自己修改調整的，不能直接使用**）

## 使用

* 該腳本在執行時會提供 `info` 和 `error` 等信息，請仔細閱讀。

### 安裝和更新 V2Ray

```
// 安裝執行檔和 .dat 資料檔
# bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh)
```

### 安裝最新發行的 geoip.dat 和 geosite.dat

```
// 只更新 .dat 資料檔
# bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-dat-release.sh)
```

### 移除 V2Ray

```
# bash <(curl -L https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh) --remove
```

### 解決問題

* 「[不安裝或更新 geoip.dat 和 geosite.dat](https://github.com/v2fly/fhs-install-v2ray/wiki/Do-not-install-or-update-geoip.dat-and-geosite.dat)」。
* 「[使用證書時權限不足](https://github.com/v2fly/fhs-install-v2ray/wiki/Insufficient-permissions-when-using-certificates)」。
* 「[從舊腳本遷移至此](https://github.com/v2fly/fhs-install-v2ray/wiki/Migrate-from-the-old-script-to-this)」。
* 「[將 .dat 文檔由 lib 目錄移動到 share 目錄](https://github.com/v2fly/fhs-install-v2ray/wiki/Move-.dat-files-from-lib-directory-to-share-directory)」。
* 「[使用 VLESS 協議](https://github.com/v2fly/fhs-install-v2ray/wiki/To-use-the-VLESS-protocol)」。

> 若您的問題沒有在上方列出，歡迎在 Issue 區提出。

**提問前請先閱讀 [Issue #63](https://github.com/v2fly/fhs-install-v2ray/issues/63)，否則可能無法得到解答並被鎖定。**

## 貢獻

請於 [develop](https://github.com/v2fly/fhs-install-v2ray/tree/develop) 分支進行，以避免對主分支造成破壞。

待確定無誤後，兩分支將進行合併。

## 新增
### 配置源
修改 /etc/apt/sources.list ：
```
#

# deb cdrom:[Debian GNU/Linux 7.3.0 _Wheezy_ - Official i386 NETINST Binary-1 20131215-03:38]/ wheezy main

#deb cdrom:[Debian GNU/Linux 7.3.0 _Wheezy_ - Official i386 NETINST Binary-1 20131215-03:38]/ wheezy main

deb http://archive.debian.org/debian/ wheezy main contrib non-free
deb-src http://archive.debian.org/debian/ wheezy main contrib non-free

deb http://archive.debian.org/debian-security/ wheezy/updates main
deb-src http://archive.debian.org/debian-security/ wheezy/updates main
```

保存后：
```bash
apt-get -o Acquire::Check-Valid-Until=false update
```
若遇到 `NO_PUBKEY` ：
```bash
apt-get install dirmngr
# apt-key adv --keyserver (keyserver.ubuntu.com) or (keyring.debian.org) --recv-keys ${KEY}
i.e.    apt-key adv --keyserver keyring.debian.org --recv-keys 7638D0442B90D010
```

### 安装基础组件：
```bash
apt-get -o Acquire::Check-Valid-Until=false update
apt-get install curl systemd python-dbus
apt-get -t wheezy install systemd-sysv
reboot
```

### 安装 V2Ray：
```bash
wget https://raw.githubusercontent.com/wdmwhh/fhs-install-v2ray-debian7/master/install-release.sh
bash install-release.sh
```

参考 [V2Ray使用和总结](http://einverne.github.io/post/2018/01/v2ray.html) 对 /usr/local/etc/v2ray/config.json 或者 l/etc/v2ray/config.json 进行修改。

开放端口
```bash
apt-get install iptables
apt-get install iptables-persistent
iptables -I INPUT -p tcp --dport 8080 -j ACCEPT
iptables-save  # iptables -L 查看是否成功开启
invoke-rc.d iptables-persistent save
```

启动 V2Ray:
```bash
systemctl enable v2ray.service
systemctl start v2ray.service
```

至此，服务端部署完成，但很可能会因为机器和软件版本出现一些异常，可能用到的分析工具 `netstat -napt （查看 V2Ray 申请端口）; systemctl list-unit-files （查看 V2Ray service 是否加入 systemd）；ps -ef | grep v2ray （查看 V2Ray 是否正常运行）`

