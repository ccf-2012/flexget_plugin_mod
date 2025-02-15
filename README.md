# Flexget Plugin Mod

在添加种子时，根据磁盘剩余空间进行删种，支持 deluge, qBittorrent
* deluge_mod: flexget deluge plugin with auto-remove
* qbittorrent_mod: flexget qBittorrent plugin with auto-remove
  
## 流程逻辑
* 每次运行 flexget execute 时，有种子符合条件要加，在加入deluge之前，会进行此逻辑判断
* 设准备新添加的种子大小为 size_new_torrent
* 计算 deluge 中当前所有正在下载的种子的剩余体积 size_left_to_complete，如果种子暂停则剩余体积不会计入
* 由 deluge 取得磁盘剩余空间 size_storage_space
* 如果 size_storage_space - size_left_to_complete > size_new_torrent 则直接加种
* 否则，对 deluge 中已完成种子，以 seeding_time 排序，逐个：
	* 假设删除种子，size_storage_space 增加种子完成的大小
	* 重新判断 size_storage_space - size_left_to_complete > size_new_torrent，成功则真实删种，并加种退出
	* 否则继续，直至所有已完成种子都假设删光
* 如果假设所有已完成种子删光仍不够空间，则不进行真实删种，也不加种，Skip退出

* Note:
1. 如果种子有手工设置部分下载并已经完成，则也会被删
2. 取种子列表会有机率出现超时失败，原因不明，如果未能取得种子列表，则跳出不加种 
3. 如果一次rss中有多个接受任务，则每个任务若接收下载，相应体积也加入判断


## 安装
* 下载 `deluge_mod.py` 到 flexget 的 plugins 目录
* create dir if not exists
```sh
mkdir -p ~/.config/flexget/plugins/
```
* save the mod into plugin dir
```sh
wget -O ~/.config/flexget/plugins/deluge_mod.py https://raw.githubusercontent.com/ccf-2012/flexget_plugin_mod/main/deluge_mod.py
```
* 或
```sh
wget -O ~/.config/flexget/plugins/qbittorrent_mod.py https://raw.githubusercontent.com/ccf-2012/flexget_plugin_mod/main/qbittorrent_mod.py
```

## 有时候需要install qbittorrent-api
```sh
cd 
source ~/.local/flexget3/bin/activate

```

```sh
pip install qbittorrent-api
```


## 配置
* 修改原有的 `config.yml` 中的 `deluge` 为 `deluge_mod`，例如：
```yaml
tasks:
  task1:
    rss: https://your_site.pt/torrentrss.php?passkey=your_rss_link
    accept_all: yes
    content_size:
      min: 1000
      max: 90000
    deluge_mod:
      port: 10006
      host: your.seed.box.ip
      username: user
      password: pass
```
* 对于 qBittorrent，则将 `qbittorrent` 改为 `qbittorrent_mod`
* 其它使用同原插件

