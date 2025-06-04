---
title: Example Title
draft: false
tags:
  - example-tag
---
 
The rest of your content lives here. You can use **Markdown** here :)
```powershell
docker build . -t rodcordeiro/quartz:latest
docker run -p 8080:8080 -v ./content:/vault rodcordeiro/quartz:latest
```
