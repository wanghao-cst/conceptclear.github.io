---
layout: post
title:  "手游fgo网易mumu模拟器自动挂机刷本脚本"
date:   2020-02-18 02:35:56
category: MobileGame
---

fgo这游戏虽然平时很长草，打一般的可以自回体拿完奖励的活动的时候也就是半天上线打三个副本就下线了，消耗的时间不多；但是当无限池或者出什么掉落材料很好的副本的时候，还是需要花费不少时间去刷的，本文对于fgo这种需要重复刷的副本进行分析并提出一种自动挂机的方法。

## 游戏机制
对于绝大多数的自由副本，一般都是由三面构成，且每面不会超过三个敌人，对于这种副本现在比较流行的方法主要是通过三面光炮解决战斗，当然想要3回合解决副本的话方法有很多，但是在活动刷副本的时候，总共六个位置自然是越多加成位越好，这样最好就是两个拐配合一个输出实现这一个光炮输出能够打3面，常用的方法分为两种：
- 基于wcba的绿卡连发体系。
- 基于弓贞冲莫类的蓝卡连发体系。

在利用绿光炮或者蓝卡光炮可以宝具3t且不需要补刀的情况之下，每次刷本需要进行的操作是一模一样的，这样就可以利用脚本进行无限刷本。进本之后的操作由于是固定的，可以按照[手游方舟指令捞UR脚本](http://conceptclear.cn/mobilegame/2018/10/07/MoblieGame-ArkOrder-blog.html)和[手游家国梦网易mumu模拟器挂机收火车及收钱脚本](http://conceptclear.cn/mobilegame/2019/10/02/homedream.html)所述方法通过设置后台模拟按键来完成刷本的工作，但是要完成连续刷本还有以下几个问题。

## 助战选择
由于现在的版本虽然可以对好友助战角色所带的礼装进行筛选，但是无法筛选助战角色，这样就需要人工来对助战角色进行选择。对于绿卡体系来说，由于自己只能带一个CBA，另一个CBA必定要从好友那边借过来，虽然说都是90级的情况下在术阶中CBA的血量是最高的，但是总会有一些好友带着为了圣杯的孔明梅林之类的角色，这样如果单纯按照血量排序的话有可能会选错助战，进本之后就会导致问题，并且有些好友的CBA技能没有练满，这样会导致伤害不足或者NP不足100从而连发失败。

针对上述问题，这里采取的办法是
- 对于没有把技能练满的好友。建议直接删除，从CBA落地至今已经过去半年之久了，中间经历了两个无限池，且CBA需要的材料在两个无限池中可以刷到，这时候还没有把技能练满完全可以删除。
- 对于喂了孔明或梅林圣杯导致血量高于CBA的好友。建议采用按键精灵自带的区域识图功能，识别助战中第一个位置的好友是否为CBA，若为CBA，则选择该助战进入下一个界面，若不是，则点击刷新好友助战，再检测第一个位置的好友是否为CBA，若不是再循环。

对于如何识别好友是否为CBA，可以通过CBA的头像进行识别，如图所示：                  
<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/MobileGame/fgo_CBA_head.jpg"></div>   

但是如上图所示由于CBA有四个阶段的头像，且每个阶段都有好友会携带，同时检测四个会加大程序的工作量从而降低程序的效率，这里更好的办法是检测CBA的宝具名，如图所示：

<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/MobileGame/fgo_CBA_noble_phantasm.jpg"></div>   

通过检测第一个助战位是否为CBA来确定是否选择到了所需要的助战，当然蓝卡队需要用狐狸或者花嫁的话只需要在此基础上进行一定修改即可，按键精灵脚本如下所示：

````
FindPic 590, 450, 970, 520, "Attachment:\CBA.bmp", 0.9, intX1, intY1      //590, 450, 970, 520分别为搜寻区域的左下角横坐标，左下角纵坐标，右上角横坐标，右上角纵坐标，0.9为相似程度
If intX1 < 0 Or intY1 < 0 Then
Do Until intX1 > 0 And intY1 > 0
	Call Plugin.Bkgnd.KeyPress(Hwnd,89)
    Delay 1000
	Call Plugin.Bkgnd.KeyPress(Hwnd,70)
    Delay 1000     //若搜寻不到，则点击助战更新再次进行搜索
    FindPic 590, 450, 970, 520, "Attachment:\CBA.bmp", 0.9, intX1, intY1
    Delay 20000     //助战更新不能一直点，为了保证程序不出问题，留出20s的余量
    Loop
End If
````

## 自动吃苹果回体
由于无限池的时候一般来说一个副本为40AP，满体力为140AP左右，由于fgo体力溢出之后是不能通过吃苹果恢复体力的，为了保证刷本的连续性，自动吃苹果刷副本是不可或缺的。                  
如下图所示，吃苹果一般有两种方法，第一种是通过点击主页面头像下方的“+”号来补充体力，第二种是当你的体力不足以打开所点开的副本时，会自动弹出吃苹果的界面，为了方便这里采取的办法为第二种。               

<div align="center"><img  src="https://github.com/conceptclear/conceptclear.github.io/raw/master/images/MobileGame/fgo_apple.jpg"></div>   

同自动选择助战一样，在每次进本之前需要判断是否需要进行补充体力的操作，这里可以通过识别副本的体力需求的颜色来进行区分（体力足够时进本的体力需求为白色，体力不足时为红色）或者通过识别区域图片来进行区分，为了留有一定余量，这里采用的是区域识图的方法，脚本如下所示：

````
FindPic 1140,360,1240,440, "Attachment:\apple.bmp", 0.9, intX, intY
Call Plugin.Bkgnd.KeyPress(Hwnd,88)      //点开副本
Delay 3000
If intX > 0 Or intY > 0 Then       //若找到图片，则吃金苹果
Call Plugin.Bkgnd.KeyPress(Hwnd,55)
Delay 2000
Call Plugin.Bkgnd.KeyPress(Hwnd,70)
Delay 3000
End If
````

## 具体脚本流程
解决掉两个比较麻烦需要进行识别的部分之后，整个脚本剩下来的部分基本上就都是重复性的工作了，这里提供我的刷本策略。
````
Hwnd = Plugin.Window.MousePoint()     //识别鼠标点击的窗口句柄，这样就可以实现后台操作不影响其他工作了
Do
FindPic 2550,1150,2650,1200, "Attachment:\apple.bmp", 0.9, intX, intY
Call Plugin.Bkgnd.KeyPress(Hwnd,88)
Delay 3000
If intX > 0 Or intY > 0 Then
Call Plugin.Bkgnd.KeyPress(Hwnd,55)
Delay 2000
Call Plugin.Bkgnd.KeyPress(Hwnd,70)
Delay 3000
End If               //这一部分为检测是否需要吃苹果

FindPic 2130, 1130, 2230, 1230, "Attachment:\CBA1.bmp", 0.9, intX1, intY1
If intX1 < 0 Or intY1 < 0 Then
Do Until intX1 > 0 And intY1 > 0
	Call Plugin.Bkgnd.KeyPress(Hwnd,89)
    Delay 1000
	Call Plugin.Bkgnd.KeyPress(Hwnd,70)
    Delay 1000
    FindPic 2130, 1130, 2230, 1230, "Attachment:\CBA1.bmp", 0.9, intX1, intY1
    Delay 20000
    Loop
End If
Call Plugin.Bkgnd.KeyPress(Hwnd,55)
Delay 2000
Call Plugin.Bkgnd.KeyPress(Hwnd,57)
Delay 1000
Delay 28000               //这一部分为检测助战是否为CBA

Call Plugin.Bkgnd.KeyPress(Hwnd,66)
Delay 500
Call Plugin.Bkgnd.KeyPress(Hwnd,49)
Delay 3000
Call Plugin.Bkgnd.KeyPress(Hwnd,69)
Delay 500
Call Plugin.Bkgnd.KeyPress(Hwnd,49)
Delay 3000
Call Plugin.Bkgnd.KeyPress(Hwnd,54)
Delay 500
Call Plugin.Bkgnd.KeyPress(Hwnd,56)
Delay 500
Call Plugin.Bkgnd.KeyPress(Hwnd,49)
Delay 3000
Call Plugin.Bkgnd.KeyPress(Hwnd,50)
Delay 2000
Call Plugin.Bkgnd.KeyPress(Hwnd,51)
Delay 500
Call Plugin.Bkgnd.KeyPress(Hwnd,52)
Delay 500
Call Plugin.Bkgnd.KeyPress(Hwnd,53)
Delay 500
Delay 27000          //打第一面

Call Plugin.Bkgnd.KeyPress(Hwnd,65)
Delay 3000
Call Plugin.Bkgnd.KeyPress(Hwnd,68)
Delay 500
Call Plugin.Bkgnd.KeyPress(Hwnd,49)
Delay 3000
Call Plugin.Bkgnd.KeyPress(Hwnd,67)
Delay 3000
Call Plugin.Bkgnd.KeyPress(Hwnd,50)
Delay 2000
Call Plugin.Bkgnd.KeyPress(Hwnd,51)
Delay 500
Call Plugin.Bkgnd.KeyPress(Hwnd,52)
Delay 500
Call Plugin.Bkgnd.KeyPress(Hwnd,53)
Delay 500
Delay 28000              //打第二面

Call Plugin.Bkgnd.KeyPress(Hwnd,70)
Delay 3000
Call Plugin.Bkgnd.KeyPress(Hwnd,71)
Delay 500
Call Plugin.Bkgnd.KeyPress(Hwnd,49)
Delay 3000
Call Plugin.Bkgnd.KeyPress(Hwnd,54)
Delay 500
Call Plugin.Bkgnd.KeyPress(Hwnd,55)
Delay 500
Call Plugin.Bkgnd.KeyPress(Hwnd,49)
Delay 3000
Call Plugin.Bkgnd.KeyPress(Hwnd,50)
Delay 2000
Call Plugin.Bkgnd.KeyPress(Hwnd,51)
Delay 500
Call Plugin.Bkgnd.KeyPress(Hwnd,52)
Delay 500
Call Plugin.Bkgnd.KeyPress(Hwnd,53)
Delay 500
Delay 27000
Call Plugin.Bkgnd.KeyPress(Hwnd,57)
Delay 1000
Call Plugin.Bkgnd.KeyPress(Hwnd,57)
Delay 1000
Call Plugin.Bkgnd.KeyPress(Hwnd,57)
Delay 1000
Call Plugin.Bkgnd.KeyPress(Hwnd,57)
Delay 1000
Call Plugin.Bkgnd.KeyPress(Hwnd,57)
Delay 1000
Delay 10000             //打第三面，并完成该副本
Loop
````

## 缺点
虽然可以实现后台自动刷副本，但是有时候会出现一定的问题。
- 由于宝具没打死需要补刀导致开环程序出现问题；
- 由于按键精灵后台识图效果不好，所以必须将网易mumu模拟器一直显示在屏幕上从而拾取屏幕区域图片；
- 由于不知道是网易mumu的问题还是按键精灵的问题，将程序最小化之后有一定几率脚本无法运行；
- 为了保证脚本的稳定性，很多地方放出了一定的时间余量，并不能以最高效率完成刷本。                       
不过尽管存在一定的问题，总体上还是很大程度上解放了双手，提高了刷本的效率。

## Reference
[1]百度百科       
[2]http://zy.anjian.com/?action-study                               
[3]http://conceptclear.cn/mobilegame/2018/10/07/MoblieGame-ArkOrder-blog.html                          
[4]http://conceptclear.cn/mobilegame/2019/10/02/homedream.html                      
