# AliyunpanACI
阿里云盘自动签到

###声明：本文内容基于原始内容进行了二次修改传播。原始内容来源为[知乎](https://zhuanlan.zhihu.com/)，作者为[小小猪](https://www.zhihu.com/people/xiao-xiao-zhu-48-51)，原始链接为[小小猪：阿里云盘自动每日签到，无需部署，无需服务器](https://zhuanlan.zhihu.com/p/629476969)。传播此内容是基于学术研究和学习目的，遵循了适用的版权规定和学术研究的合理使用原则。  

执行思路：使用金山文档的每日定时任务，执行阿里云盘签到接口。
无需部署，无需服务器，每个月更新一次token。  

###  
[需要用到的模板文件，另存为副本到自己的账户中](https://kdocs.cn/l/ceQR4HpZ6op1)
  
将这段代码复制到[效率>高级开发>AirScript]中
```
varmyDate=newDate();// 创建一个表示当前时间的 Date 对象vardata_time=myDate.toLocaleDateString();// 获取当前日期的字符串表示functionsleep(d){for(vart=Date.now();Date.now()-t<=d;);// 使程序暂停执行一段时间}functionlog(message){console.log(message);// 打印消息到控制台// TODO: 将日志写入文件}vartokenColumn="A";// 设置列号变量为 "A"varsignInColumn="B";// 设置列号变量为 "B"varrewardColumn="C";// 设置列号变量为 "C"varemailColumn="F";// 设置列号变量为 "F"varsendEmailColumn="G";// 设置列号变量为 "G"varresultColumn="J";// 设置列号变量为 "J"for(letrow=2;row<=20;row++){// 循环遍历从第 2 行到第 20 行的数据varrefresh_token=Application.Range(tokenColumn+row).Text;// 获取指定单元格的值varsflq=Application.Range(signInColumn+row).Text;// 获取指定单元格的值varsflqReward=Application.Range(rewardColumn+row).Text;// 获取指定单元格的值varjsyx=Application.Range(emailColumn+row).Text;// 获取指定单元格的值varsendEmail=Application.Range(sendEmailColumn+row).Text;// 获取指定单元格的值varcustomEmailResult=Application.Range(resultColumn+row).Text;// 获取指定单元格的值varemailConfigured=Application.Range("J1").Text;// 获取指定单元格的值varzdy_host=Application.Range("J2").Text;// 获取指定单元格的值varzdy_post=parseInt(Application.Range("J3").Text);// 获取指定单元格的值并转换为整数varzdy_username=Application.Range("J4").Text;// 获取指定单元格的值varzdy_pasd=Application.Range("J5").Text;// 获取指定单元格的值if(sflq=="是"){// 如果“是否签到”为“是”if(refresh_token!=""){// 如果刷新令牌不为空// 发起网络请求-获取tokenletdata=HTTP.post("https://auth.aliyundrive.com/v2/account/token",JSON.stringify({"grant_type":"refresh_token","refresh_token":refresh_token}));data=data.json();// 将响应数据解析为 JSON 格式varaccess_token=data['access_token'];// 获取访问令牌varphone=data["user_name"];// 获取用户名if(access_token==undefined){// 如果访问令牌未定义log("单元格【"+tokenColumn+row+"】内的token值错误，程序执行失败，请重新复制正确的token值");continue;// 跳过当前行的后续操作}try{varaccess_token2='Bearer '+access_token;// 构建包含访问令牌的请求头// 签到letdata2=HTTP.post("https://member.aliyundrive.com/v1/activity/sign_in_list",JSON.stringify({"_rx-s":"mobile"}),{headers:{"Authorization":access_token2}});data2=data2.json();// 将响应数据解析为 JSON 格式varsignin_count=data2['result']['signInCount'];// 获取签到次数varlogMessage="账号："+phone+" - 签到成功，本月累计签到 "+signin_count+" 天";varrewardMessage="";if(sflqReward=="是"){// 如果“是否领取奖励”为“是”if(sflq=="是"){// 如果“是否签到”为“是”try{// 领取奖励letdata3=HTTP.post("https://member.aliyundrive.com/v1/activity/sign_in_reward?_rx-s=mobile",JSON.stringify({"signInDay":signin_count}),{headers:{"Authorization":access_token2}});data3=data3.json();// 将响应数据解析为 JSON 格式varrewardName=data3["result"]["name"];// 获取奖励名称varrewardDescription=data3["result"]["description"];// 获取奖励描述rewardMessage=" "+rewardName+" - "+rewardDescription;}catch(error){if(error.response&&error.response.data&&error.response.data.error){varerrorMessage=error.response.data.error;// 获取错误信息if(errorMessage.includes(" - 今天奖励已领取")){rewardMessage=" - 今天奖励已领取";log("账号："+phone+" - "+rewardMessage);}else{log("账号："+phone+" - 奖励领取失败："+errorMessage);}}else{log("账号："+phone+" - 奖励领取失败");}}}else{rewardMessage=" - 奖励待领取";}}else{rewardMessage=" - 奖励待领取";}log(logMessage+rewardMessage);if(sendEmail=="是"){// 如果“是否发送邮件”为“是”try{letmailer;if(customEmailResult=="是"){// 如果“是否自定义邮箱”为“是”varcustomEmail=Application.Range(resultColumn+row).Text;// 获取指定单元格的值if(emailConfigured==="是"){// 如果配置了自定义邮箱mailer=SMTP.login({host:zdy_host,port:zdy_post,username:zdy_username,password:zdy_pasd,secure:true});mailer.send({from:"阿里云盘签到<"+zdy_username+">",to:customEmail,subject:"阿里云盘签到通知 - "+data_time,text:logMessage+rewardMessage});}else{// 如果未配置自定义邮箱，默认使用示例邮箱mailer=SMTP.login({host:"smtp.163.com",port:465,username:"fs8484848@163.com",password:"QADSEMPKDHDAVWVD",secure:true});mailer.send({from:"阿里云盘签到",to:customEmail,subject:"阿里云盘签到通知 - "+data_time,text:logMessage+rewardMessage});}log("账号："+phone+" - 已发送邮件至："+customEmail);}else{// 如果“是否自定义邮箱”为“否”if(emailConfigured==="是"){// 如果配置了自定义邮箱mailer=SMTP.login({host:zdy_host,port:zdy_post,username:zdy_username,password:zdy_pasd,secure:true});mailer.send({from:"阿里云盘签到<"+zdy_username+">",to:jsyx,subject:"阿里云盘签到通知 - "+data_time,text:logMessage+rewardMessage});}else{// 如果未配置自定义邮箱，默认使用示例邮箱mailer=SMTP.login({host:"smtp.163.com",port:465,username:"fs8484848@163.com",password:"QADSEMPKDHDAVWVD",secure:true});mailer.send({from:"阿里云盘签到",to:jsyx,subject:"阿里云盘签到通知 - "+data_time,text:logMessage+rewardMessage});}log("账号："+phone+" - 已发送邮件至："+jsyx);}}catch(error){log("账号："+phone+" - 发送邮件失败："+error);}}}catch{log("单元格【"+tokenColumn+row+"】内的token签到失败");continue;// 跳过当前行的后续操作}}else{log("账号："+phone+" 不签到");}}}varcurrentDate=newDate();// 创建一个表示当前时间的 Date 对象varcurrentDay=currentDate.getDate();// 获取当前日期的天数varlastDayOfMonth=newDate(currentDate.getFullYear(),currentDate.getMonth()+1,0).getDate();// 获取当月的最后一天的日期if(currentDay===lastDayOfMonth){// 如果当前日期是当月的最后一天for(letrow=2;row<=20;row++){// 循环遍历从第 2 行到第 20 行的数据varsflq=Application.Range(signInColumn+row).Text;// 获取指定单元格的值varsflqReward=Application.Range(rewardColumn+row).Text;// 获取指定单元格的值if(sflq==="是"&&sflqReward==="是"){// 如果“是否签到”和“是否领取奖励”均为“是”varrefresh_token=Application.Range(tokenColumn+row).Text;// 获取指定单元格的值varjsyx=Application.Range(emailColumn+row).Text;// 获取指定单元格的值varphone="账号："+phone;// 构建账号信息字符串if(refresh_token!==""){// 如果刷新令牌不为空// 发起网络请求-获取tokenletdata=HTTP.post("https://auth.aliyundrive.com/v2/account/token",JSON.stringify({"grant_type":"refresh_token","refresh_token":refresh_token}));data=data.json();// 将响应数据解析为 JSON 格式varaccess_token=data['access_token'];// 获取访问令牌if(access_token===undefined){// 如果访问令牌未定义log("单元格【"+tokenColumn+row+"】内的token值错误，程序执行失败，请重新复制正确的token值");continue;// 跳过当前行的后续操作}try{varaccess_token2='Bearer '+access_token;// 构建包含访问令牌的请求头// 领取奖励letdata4=HTTP.post("https://member.aliyundrive.com/v1/activity/sign_in_reward?_rx-s=mobile",JSON.stringify({"signInDay":lastDayOfMonth}),{headers:{"Authorization":access_token2}});data4=data4.json();// 将响应数据解析为 JSON 格式varclaimStatus=data4["result"]["status"];// 获取奖励状态varday=lastDayOfMonth;// 获取最后一天的日期if(claimStatus==="CLAIMED"){log("账号："+phone+" - 第 "+day+" 天奖励领取成功");}else{log("账号："+phone+" - 第 "+day+" 天奖励领取失败");}}catch{log("单元格【"+tokenColumn+row+"】内的token签到失败");continue;// 跳过当前行的后续操作}}else{log("账号："+phone+" 不签到");}}}log("自动领取未领取奖励完成。");}
```
![C](https://github.com/yohototo/AliyunpanACI/assets/35169273/da0512b6-4dbd-4348-9261-e3b973cc4836)

