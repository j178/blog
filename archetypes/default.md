---
title: {{ replace .Name "-" " " | title }}
slug: {{ slicestr .Name 11 }}
date: {{ .Date }}
draft: true
---
