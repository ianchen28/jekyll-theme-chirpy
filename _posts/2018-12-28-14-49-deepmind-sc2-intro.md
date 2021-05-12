---
layout: post
title: DeepMind pysc2 简介和部署记录
updated: 2018-12-28 14:51
---

* TOC
{:toc}

## pysc2简介

DeepMind于2017年8月公布的SC2LE (StarCraft II Learning Environment)的python部分，将SC II的机器学习官方API包括一个匿名游戏replay数据集（[链接](https://github.com/Blizzard/s2client-proto)）以Python RL环境的形式暴露出来（[blog](https://github.com/Blizzard/s2client-proto)）

可以简单用一句脚本安装

```bash
pip install pysc2
```

将相关地图（[链接](https://github.com/Blizzard/s2client-proto)）下载放入StarCraftII/Maps/中即可运行

例如简单随机agent

```bash
python -m pysc2.bin.agent --map Simple64
```

更复杂的agent参见git链接

pysc2还包含了一些mini-games，即一些小型任务而非正式战役

## 相关资源

统计机器学习方法的教程范例（[链接](https://pythonprogramming.net/starcraft-ii-ai-python-sc2-tutorial/)），该范例是基于简化版的sc2，而不是DeepMind的pysc2，但是从零开始可以了解API结构

基于pysc2的RL教程（[链接](https://chatbotslife.com/building-a-basic-pysc2-agent-b109cde1477c)）

DeepMind pysc2 （[项目链接](https://github.com/deepmind/pysc2)）

commandcenter （[项目链接](https://github.com/davechurchill/commandcenter)），支持SC和SC II
