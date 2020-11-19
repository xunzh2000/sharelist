# ShareList

ShareList 是一个易用的网盘工具，支持快速挂载 GoogleDrive、OneDrive ，可通过插件扩展功能。

## 目录
* [功能说明](#功能说明) 
* [安装](#安装) 
* [使用示例](#使用示例)   
  * [挂载本地文件](#挂载本地文件) 
  * [挂载天翼云、和彩云盘](#挂载天翼云、和彩云盘) 
  * [挂载WebDAV](#挂载WebDAV) 
  * [虚拟目录](#虚拟目录) 
  * [虚拟文件](#虚拟文件) 
  * [目录加密](#目录加密) 
  * [流量中转](#流量中转) 
  * [忽略文件类型](#忽略文件类型) 
  * [文件预览](#文件预览) 
  * [显示README](#显示README) 
  * [负载均衡](#负载均衡) 
  * [Nginx/Caddy反代注意事项](#Nginx/Caddy反代注意事项) 
* [插件开发](#插件开发) 


## 功能说明
- 多种网盘系统快速挂载。 
- 支持虚拟目录和虚拟文件。 
- 支持目录加密。 
- 插件机制。 
- 国际化支持。  
- WebDAV导出。  

### 挂载天翼云、和彩云盘
ShareList 支持账号密码挂载，所以你不用为没有sk而担心挂载不了。

1.账号密码挂载（Cookie方式） 由drive.189cloud.js插件实现。 挂载标示：ctcc 挂载内容：
//用户名/初始文件夹ID?password=密码 / 建议填写/，ShareList将自动开启挂载向导，按指示填写用户名密码即可。 登录天翼云盘网页版，点击相应的目录，文件夹id就在网址里面。
2. API方式挂载
由drive.189cloud.api.js插件实现。
挂载标示：ctc
挂载内容：   
    //应用ID/初始文件夹ID?app_secret=应用机钥&redirect_uri=回调地址&access_token=access_token   
    /
建议填写/，ShareList将自动开启挂载向导，按指示操作即可。
注意：access_token每隔30天需手动更新一次，到期前24小时内访问对应路径时会有更新提示。
注意：如果你天翼云登录成功，就不要随意修改，因为账号密码有概率出现验证码，如果出现你就登录不上，也就没法使用这个插件了。
天翼云，和蓝奏云效果还是不错的，不过天翼云会大概率遇到验证码，所以有时候会挂载不了。
注意：dplayer无法跨域播放，需要启用中转模式。或者你只能启用默认播放器。设置中转很方便，后台折腾即可。

 挂载和彩云

路径填 /即可。


### 挂载本地文件
由[drive.fs.js](app/plugins/drive.fs.js)插件实现。  
```
挂载标示：fs   
挂载内容：文件路径。
```
**注意：统一使用unix风格路径，例如 windows D盘 为 ```/d/```。**    


### 挂载WebDAV
由[drive.webdav.js](plugins/drive.webdav.js)插件实现，用于访问WebDAV服务。  
```
挂载标示：webdav  
挂载路径：  
  https://webdavserver.com:1222/path   
  https://username:password@webdavserver.com:1222/path   
  https://username:password@webdavserver.com:1222/?acceptRanges=none
```
**注意：若服务端不支持断点续传，需追加```acceptRanges=none```** 

### 虚拟目录
在需创建虚拟目录处新建```目录名.d.ln```文件。 其内容为```挂载标识:挂载路径```。   
指向本地```/root```的建虚拟目录  
```   
fs:/root 
``` 

指向GoogleDrive的某个共享文件夹虚拟目录   
```
gd:0BwfTxffUGy_GNF9KQ25Xd0xxxxxxx 
```  
系统内置了一种单文件虚拟目录系统，使用yaml构建，以```sld```作为后缀保存。参考[example/ShareListDrive.sld](example/sharelist_drive.sld)。 


### 虚拟文件
与虚拟目录类似，目标指向具体文件。  
在需创建虚拟文件处新建```文件名.后缀名.ln```文件。 其内容为```挂载标识:挂载路径```。 
如：创建一个```ubuntu_18.iso```的虚拟文件，请参考[example/linkTo_download_ubuntu_18.iso.ln](example)。 
  

### 目录加密
在需加密目录内新建 ```.passwd``` 文件，```type```为验证方式，```data```为验证内容。  
目前只支持用户名密码对加密（由[auth.basic](app/plugins/auth.basic.js)插件实现）。
例如：    
```yaml
type: basic 
data: 
  - user1:111111 
  - user2:aaaaaa 
``` 

```user1```用户可使用密码```111111```验证，```user2```用户可使用密码```aaaaaa```验证。请参考[example/secret_folder/.passwd](example)。 

### 流量中转
后台管理，常规设置，将```中转（包括预览）```设为启用即可实现中转代理。

### 负载均衡
由[drive.lb.js](app/plugins/drive.lb.js)插件实现。用于将请求转发到多个对等的网盘。       
```
挂载标示：lb
挂载路径：  
  用;分割多个路径地址  
``` 

例如，已经在```http://localhost/a```和```http://localhost/b```路径上挂载了内容相同的两个网盘，需要将两者的请求其合并到```http://localhost/c```路径下，在后台虚拟路径处，选择LoadBalancer类型，挂载路径填写为```/a;/b```即可。 

**注意：负载目录建立后，其目标目录将被自动隐藏（管理员模式可见）。**   

### 忽略文件类型 
后台管理，常规设置，```忽略文件类型```可定义忽略的文件类型。例如忽略图片：```jpg,png,gif,webp,bmp,jpeg```。  
### 显示README
后台管理，常规设置，将```显示README.md内容```设为启用，当前目录包含```README.md```时，将自动显示在页面。

### 文件预览
后台管理，常规设置，将```详情预览```设为启用即可对特定文件进行预览。目前支持：   

#### 文档类
由[preview.document](plugins/drive.document.js)插件实现，可预览md、word、ppt、excel。

#### 多媒体
由[preview.media](plugins/drive.media.js)插件实现，可预览图片、音频、视频提供。  
后台管理，插件设置，```支持预览的视频后缀```可定义可预览视频类型。  

#### Torrent  
由[preview.torrent](plugins/drive.torrent.js)插件实现，为种子文件提供在线预览。

### Nginx/Caddy反代注意事项
使用反代时，请添加以下配置。  
Nginx   
```ini
  proxy_set_header Host  $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
```   
Caddy   
```ini
  header_upstream Host {host}
  header_upstream X-Real-IP {remote}
  header_upstream X-Forwarded-For {remote}
  header_upstream X-Forwarded-Proto {scheme}
```

## 插件开发
待完善   


## 安装
### Shell
```bash
bash install.sh
```
远程安装 / Netinstall
```bash
wget --no-check-certificate -qO-  https://raw.githubusercontent.com/reruin/sharelist/master/netinstall.sh | bash
```
更新 / Update
```bash
bash update.sh
```  

### Docker support
```bash
docker build -t yourname/sharelist .

docker run -d -v /etc/sharelist:/app/cache -p 33001:33001 --name="sharelist" yourname/sharelist
```

OR

```bash
docker-compose up
```

访问 `http://localhost:33001` 
WebDAV 目录 `http://localhost:33001/webdav` 


### Heroku

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy?template=https://github.com/reruin/sharelist-heroku)
