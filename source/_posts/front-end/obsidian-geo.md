---
title: obsidian-geo
layout: post
categories: front-end
date: 2024-11-24 22:30:55
updated: 2024-11-24 22:30:55
description:
---

一直想对笔记的地理位置进行可视化的查看，社区里面 https://github.com/esm7/obsidian-map-view 这个插件实现了一部分，但是初始化的地理位置录入还是笔记麻烦。

琢磨了下，实现了部分，因为比较个人化，不准备上架商店。

代码仓库：https://github.com/Yaob1990/obsidian-geo
# mac

mac 是比较容易的，使用的是 mac 的捷径，通过天气的接口获取到的地理位置。（pc 写的图片后补）

# pc

```ts
  const command = `

          Add-Type -AssemblyName System.Device;

          $GeoWatcher = New-Object System.Device.Location.GeoCoordinateWatcher;

          $GeoWatcher.Start();

          Start-Sleep -Milliseconds 1000;

          $status = $GeoWatcher.Status;

          if ($status -eq 'Ready') {

            $lat = $GeoWatcher.Position.Location.Latitude;

            $lon = $GeoWatcher.Position.Location.Longitude;

            Write-Output "$lat,$lon"

          } else {

            Write-Error "无法获取位置信息: $status"

          }

        `;
```

这段代码在 powershell 里面确实能够获取到地理位置，但是在 obsidian 的 Electron 沙盒里面无法获取...
考虑到我写内容，pc 是固定在家里面的，所以，我判断了平台是 windows 之后，直接写死了返回值...