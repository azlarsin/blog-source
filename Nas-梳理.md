title: Nas 梳理
author: azlar
date: '2023-02-07 12:47:52'
tags: []

---

<!-- desc -->

## 配置
TS - 464C。

HDD1 16t 作为影音盘，HDD2 12t 作为日常处理盘。
ssd1 为系统盘。
ssd2 为缓存盘。

## 迁移背景
1. 12t 装满影音、入 nas 需要**格式化**。
2. 12t Mac Photos 需要一步步导出到文件。


### 流程
1. 导出所有照片为单片，移动到 HDD1？
2. 冷备份 Mac Photos，移动到 HDD1？
3. 移动所有的 videos 到 HDD1
4. 删除 （相机上传），这一步会释放空间。这一步比较关键。
5. 移动零碎文件到 HDD1。（这里，会占满 hdd1）
6. 拆解 HDD2
7. 入 nas。格式化 HDD2
8. 设置工作目录到 HDD2
9. 迁移 videos 到 HDD2