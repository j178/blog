---
title: 模拟登陆 12306
tags: Python
---

- 验证码URL
  https://kyfw.12306.cn/otn/passcodeNew/getPassCodeNew?module=login&rand=sjrand

- 验证码提交: https://kyfw.12306.cn/otn/passcodeNew/checkRandCodeAnsyn

  randCode: 11,17 (鼠标点击的坐标, 按点击顺序排列)
  rand: sjrand

- 登陆 https://kyfw.12306.cn/otn/login/loginAysnSuggest
	loginUserDTO.user_name: 
	userDTO.password: 
	randCode: 253,59,60,125