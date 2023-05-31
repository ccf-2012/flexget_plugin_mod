# Flexget Plugin Mod

* deluge_mod: flexget deluge plugin with auto-remove, 在添加种子时，根据磁盘剩余空间进行删种

## 流程逻辑
* 设准备新添加的种子大小为 size_new_torrent
* 计算 deluge 中当前所有正在下载的种子的剩余体积 size_left_to_complete
* 如果磁盘的剩余空间 size_storage_space - size_left_to_complete > size_new_torrent 则直接加种
* 否则，对 deluge 中已完成种子，以 seeding_time 排序，逐个：
	* 假设删除种子，size_storage_space 增加种子完成的大小
	* 重新判断 size_storage_space - size_left_to_complete > size_new_torrent，成功则真实删种，并加种退出
	* 否则继续，直至所有已完成种子都假设删光
* 如果假设所有已完成种子删光仍不够空间，则不进行删种，也不加种，Skip退出

* Note:
1. 每次运行 flexget execute 时，有任务要加时，会进行此逻辑判断
2. 磁盘剩余空间由 deluge 取得
3. 如果种子暂停，因不是在下载，所以其剩余体积不会加入 size_left_to_complete
4. 如果种子有手工设置部分下载并已经完成，则也会被删


## 安装
* 下载 `deluge_mod.py` 到 flexget 的 plugins 目录
```sh
wget -P ~/.config/flexget/plugins/ https://github.com/ccf-2012/flexget_plugin_mod/blob/main/deluge_mod.py
```

## 配置
* 修改原有的 `config.yml` 中的 `deluge` 为 `deluge_mod`，例如：
```yaml
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
* 其它使用同原 deluge 插件

