---
<%*
// 1. 获取当前日期时间，格式为 YYYY-MM-DD-HH-mm-ss- 
const datePrefix = tp.date.now("YYYY-MM-DD-HH-mm-ss-");

// 2. 获取你输入的标题（如果没有输入，默认为"未命名"） 
const title = await tp.system.prompt("输入文章标题");

// 3. 执行重命名：日期前缀 + 标题 + .md 
await tp.file.rename(`${datePrefix}${title}`);
-%>
title : <% title %>
date : <% tp.date.now("YYYY-MM-DD HH:mm:ss") %>
category : <% tp.system.prompt("输入分类") %>
tags : 
---
