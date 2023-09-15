# AliyunpanACI
阿里云盘自动签到

**声明：本文内容基于原始内容进行了二次修改传播。原始内容来源为[知乎](https://zhuanlan.zhihu.com/)，作者为[小小猪](https://www.zhihu.com/people/xiao-xiao-zhu-48-51)，原始链接为[小小猪：阿里云盘自动每日签到，无需部署，无需服务器](https://zhuanlan.zhihu.com/p/629476969)。传播此内容是基于学术研究和学习目的，遵循了适用的版权规定和学术研究的合理使用原则。**  

执行思路：使用金山文档的每日定时任务，执行阿里云盘签到接口。
无需部署，无需服务器，每个月更新一次token。  

###  
[需要用到的模板文件，另存为副本到自己的账户中https://kdocs.cn/l/ceQR4HpZ6op1](https://kdocs.cn/l/ceQR4HpZ6op1)
  
将这段代码复制到 [效率>高级开发>AirScript] 中
```
varmyDate=newDate();// 创建一个表示当前时间的 Date 对象vardata_time=myDate.toLocaleDateString();// 获取当前日期的字符串表示functionsleep(d){for(vart=Date.now();Date.now()-t<=d;);// 使程序暂停执行一段时间}functionlog(message){console.log(message);// 打印消息到控制台// TODO: 将日志写入文件}vartokenColumn="A";// 设置列号变量为 "A"varsignInColumn="B";// 设置列号变量为 "B"varrewardColumn="C";// 设置列号变量为 "C"varemailColumn="F";// 设置列号变量为 "F"varsendEmailColumn="G";// 设置列号变量为 "G"varresultColumn="J";// 设置列号变量为 "J"for(letrow=2;row<=20;row++){// 循环遍历从第 2 行到第 20 行的数据varrefresh_token=Application.Range(tokenColumn+row).Text;// 获取指定单元格的值varsflq=Application.Range(signInColumn+row).Text;// 获取指定单元格的值varsflqReward=Application.Range(rewardColumn+row).Text;// 获取指定单元格的值varjsyx=Application.Range(emailColumn+row).Text;// 获取指定单元格的值varsendEmail=Application.Range(sendEmailColumn+row).Text;// 获取指定单元格的值varcustomEmailResult=Application.Range(resultColumn+row).Text;// 获取指定单元格的值varemailConfigured=Application.Range("J1").Text;// 获取指定单元格的值varzdy_host=Application.Range("J2").Text;// 获取指定单元格的值varzdy_post=parseInt(Application.Range("J3").Text);// 获取指定单元格的值并转换为整数varzdy_username=Application.Range("J4").Text;// 获取指定单元格的值varzdy_pasd=Application.Range("J5").Text;// 获取指定单元格的值if(sflq=="是"){// 如果“是否签到”为“是”if(refresh_token!=""){// 如果刷新令牌不为空// 发起网络请求-获取tokenletdata=HTTP.post("https://auth.aliyundrive.com/v2/account/token",JSON.stringify({"grant_type":"refresh_token","refresh_token":refresh_token}));data=data.json();// 将响应数据解析为 JSON 格式varaccess_token=data['access_token'];// 获取访问令牌varphone=data["user_name"];// 获取用户名if(access_token==undefined){// 如果访问令牌未定义log("单元格【"+tokenColumn+row+"】内的token值错误，程序执行失败，请重新复制正确的token值");continue;// 跳过当前行的后续操作}try{varaccess_token2='Bearer '+access_token;// 构建包含访问令牌的请求头// 签到letdata2=HTTP.post("https://member.aliyundrive.com/v1/activity/sign_in_list",JSON.stringify({"_rx-s":"mobile"}),{headers:{"Authorization":access_token2}});data2=data2.json();// 将响应数据解析为 JSON 格式varsignin_count=data2['result']['signInCount'];// 获取签到次数varlogMessage="账号："+phone+" - 签到成功，本月累计签到 "+signin_count+" 天";varrewardMessage="";if(sflqReward=="是"){// 如果“是否领取奖励”为“是”if(sflq=="是"){// 如果“是否签到”为“是”try{// 领取奖励letdata3=HTTP.post("https://member.aliyundrive.com/v1/activity/sign_in_reward?_rx-s=mobile",JSON.stringify({"signInDay":signin_count}),{headers:{"Authorization":access_token2}});data3=data3.json();// 将响应数据解析为 JSON 格式varrewardName=data3["result"]["name"];// 获取奖励名称varrewardDescription=data3["result"]["description"];// 获取奖励描述rewardMessage=" "+rewardName+" - "+rewardDescription;}catch(error){if(error.response&&error.response.data&&error.response.data.error){varerrorMessage=error.response.data.error;// 获取错误信息if(errorMessage.includes(" - 今天奖励已领取")){rewardMessage=" - 今天奖励已领取";log("账号："+phone+" - "+rewardMessage);}else{log("账号："+phone+" - 奖励领取失败："+errorMessage);}}else{log("账号："+phone+" - 奖励领取失败");}}}else{rewardMessage=" - 奖励待领取";}}else{rewardMessage=" - 奖励待领取";}log(logMessage+rewardMessage);if(sendEmail=="是"){// 如果“是否发送邮件”为“是”try{letmailer;if(customEmailResult=="是"){// 如果“是否自定义邮箱”为“是”varcustomEmail=Application.Range(resultColumn+row).Text;// 获取指定单元格的值if(emailConfigured==="是"){// 如果配置了自定义邮箱mailer=SMTP.login({host:zdy_host,port:zdy_post,username:zdy_username,password:zdy_pasd,secure:true});mailer.send({from:"阿里云盘签到<"+zdy_username+">",to:customEmail,subject:"阿里云盘签到通知 - "+data_time,text:logMessage+rewardMessage});}else{// 如果未配置自定义邮箱，默认使用示例邮箱mailer=SMTP.login({host:"smtp.163.com",port:465,username:"fs8484848@163.com",password:"QADSEMPKDHDAVWVD",secure:true});mailer.send({from:"阿里云盘签到",to:customEmail,subject:"阿里云盘签到通知 - "+data_time,text:logMessage+rewardMessage});}log("账号："+phone+" - 已发送邮件至："+customEmail);}else{// 如果“是否自定义邮箱”为“否”if(emailConfigured==="是"){// 如果配置了自定义邮箱mailer=SMTP.login({host:zdy_host,port:zdy_post,username:zdy_username,password:zdy_pasd,secure:true});mailer.send({from:"阿里云盘签到<"+zdy_username+">",to:jsyx,subject:"阿里云盘签到通知 - "+data_time,text:logMessage+rewardMessage});}else{// 如果未配置自定义邮箱，默认使用示例邮箱mailer=SMTP.login({host:"smtp.163.com",port:465,username:"fs8484848@163.com",password:"QADSEMPKDHDAVWVD",secure:true});mailer.send({from:"阿里云盘签到",to:jsyx,subject:"阿里云盘签到通知 - "+data_time,text:logMessage+rewardMessage});}log("账号："+phone+" - 已发送邮件至："+jsyx);}}catch(error){log("账号："+phone+" - 发送邮件失败："+error);}}}catch{log("单元格【"+tokenColumn+row+"】内的token签到失败");continue;// 跳过当前行的后续操作}}else{log("账号："+phone+" 不签到");}}}varcurrentDate=newDate();// 创建一个表示当前时间的 Date 对象varcurrentDay=currentDate.getDate();// 获取当前日期的天数varlastDayOfMonth=newDate(currentDate.getFullYear(),currentDate.getMonth()+1,0).getDate();// 获取当月的最后一天的日期if(currentDay===lastDayOfMonth){// 如果当前日期是当月的最后一天for(letrow=2;row<=20;row++){// 循环遍历从第 2 行到第 20 行的数据varsflq=Application.Range(signInColumn+row).Text;// 获取指定单元格的值varsflqReward=Application.Range(rewardColumn+row).Text;// 获取指定单元格的值if(sflq==="是"&&sflqReward==="是"){// 如果“是否签到”和“是否领取奖励”均为“是”varrefresh_token=Application.Range(tokenColumn+row).Text;// 获取指定单元格的值varjsyx=Application.Range(emailColumn+row).Text;// 获取指定单元格的值varphone="账号："+phone;// 构建账号信息字符串if(refresh_token!==""){// 如果刷新令牌不为空// 发起网络请求-获取tokenletdata=HTTP.post("https://auth.aliyundrive.com/v2/account/token",JSON.stringify({"grant_type":"refresh_token","refresh_token":refresh_token}));data=data.json();// 将响应数据解析为 JSON 格式varaccess_token=data['access_token'];// 获取访问令牌if(access_token===undefined){// 如果访问令牌未定义log("单元格【"+tokenColumn+row+"】内的token值错误，程序执行失败，请重新复制正确的token值");continue;// 跳过当前行的后续操作}try{varaccess_token2='Bearer '+access_token;// 构建包含访问令牌的请求头// 领取奖励letdata4=HTTP.post("https://member.aliyundrive.com/v1/activity/sign_in_reward?_rx-s=mobile",JSON.stringify({"signInDay":lastDayOfMonth}),{headers:{"Authorization":access_token2}});data4=data4.json();// 将响应数据解析为 JSON 格式varclaimStatus=data4["result"]["status"];// 获取奖励状态varday=lastDayOfMonth;// 获取最后一天的日期if(claimStatus==="CLAIMED"){log("账号："+phone+" - 第 "+day+" 天奖励领取成功");}else{log("账号："+phone+" - 第 "+day+" 天奖励领取失败");}}catch{log("单元格【"+tokenColumn+row+"】内的token签到失败");continue;// 跳过当前行的后续操作}}else{log("账号："+phone+" 不签到");}}}log("自动领取未领取奖励完成。");}
```
从浏览器中获取 refresh_token 的值：浏览器登录阿里云盘  
![image](https://github.com/yohototo/AliyunpanACI/assets/35169273/cb1ad658-9f46-4fbe-854a-eeab094bdcf4)
将refresh_token 的值复制到表格中，（A2-A20）可以写入多个账号的refresh_token  
![image](https://github.com/yohototo/AliyunpanACI/assets/35169273/64555235-ff96-42fa-a61c-2a0a08dff67b)
10、填写表格内容：

10.1：填写是否领取奖励（是的话会自动领取签到奖励，为否的话只签到，需要用的时候自己手动领取签到奖励）

10.2：填写是否发送邮箱通知，发送邮箱通知的话，要写入接收邮箱的地址，不发送就不用写

10.3：填写是否自定义发送邮箱，这里推荐会弄SMTP的人自己填写自己的发送邮箱配置（发送和接收的邮箱可以相同），不会弄的人就写否或者不写就好了，我在代码里写了一个发送邮箱，但是邮箱有每日限制发送数量，可能会有接收不到邮件的情况。
![image](https://github.com/yohototo/AliyunpanACI/assets/35169273/202364be-cf95-4b10-8823-8301e30d5657)
脚本中，点击上方的【保存】按钮，再点击【运行】按钮  
![image](https://github.com/yohototo/AliyunpanACI/assets/35169273/10bc3130-de0f-4415-a2da-accad3f25c99)
关闭代码编辑框，点击【效率】-【高级开发】-【定时任务】，点击【创建任务】，
![image](https://github.com/yohototo/AliyunpanACI/assets/35169273/ec45b177-c729-4ce2-b670-4b17ca1bade2)
设置每天运行的时间，选择刚刚选择的脚本，保存，大功告成  
![image](https://github.com/yohototo/AliyunpanACI/assets/35169273/218a62d0-3508-4081-8b93-14e4ae7b2613)


如何获取自己的SMTP  
这里我以网易邮箱为例：  
打开网易官网：https://mail.163.com/，登录账号  
![image](https://github.com/yohototo/AliyunpanACI/assets/35169273/ae910529-41dd-46e2-8899-55a7ca4b1c16)
![image](https://github.com/yohototo/AliyunpanACI/assets/35169273/4a5808f4-229f-434d-b24b-b5ce2f0f8da9)

