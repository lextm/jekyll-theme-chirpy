---
layout: post
title: "IIS迷宫之消失的证书"
description: "这篇文章讲述了在 IIS 中配置 SSL 证书出现证书消失问题的时候应该如何应对，以及这个问题的追踪中涉及到的证书、IIS 相关知识"
tags: IIS Windows
permalink: /iis迷宫之消失的证书-558d58aff8f5
excerpt_separator: <!--more-->
---

在网站开发和测试工作中，总有一些场景像迷宫一样繁杂，以至于一个不小心就会如无头苍蝇到处乱撞。今天这个案例便起因于一个小设置，但没有恶补下X509证书基础知识的看官们恐怕尚且需要些时间来消化啊。
<!--more-->

“老大，总监刚才说我们的官方网站要加装证书启用HTTPS！！！怎么搞啊？”

小张每次都是这样大着个嗓门，完全沉不住气，即使我这副耳机有主动降噪功能也拿他毫无办法。

“怎么搞？上网找个CA申请一张呗，哪个便宜用哪个。你自行股沟一下啦。”

每逢这种场面，我还是蛮放心大胆让手下人自己去钻研，毕竟他们年轻力壮，就是不幸要加个班也从来不抱怨。嗯，加班有晚餐补贴对于这些小朋友还是蛮有吸引力的。

所以我就放心的摘掉耳机，关了电脑下班走人了，留下小张一个人在屏幕前抓耳挠腮。

第二天，本想着小张同学一切搞定该找我报销买证书的钱了，结果远远瞧着他一上午都在电脑前磨洋工，也不知道是不是又在捯饬私活。懒得管他。

要吃午饭了，我一看没带钱包！！！只好硬着头皮走向他的工位，看看这个中午十二点唯一还在办公室的小年轻能不能借点零钱给我吃个麦叔叔。

“忙啥哪？怎么还不去吃饭？”

“老大。。。”那小子居然一脸沮丧，无精打采。

“怎么了？昨天。。。”我回头看看四下无人，“搞私活又加夜班了？”

“天地良心啊。。我真没干私活！”这小子还嘴硬，不过手指指电脑屏幕，让我看他似乎一直开着的IIS Manager。

“老大，你来看下，这个证书申请回来死活装不好，一定是微软产品的bug！”

“哦？”我最烦手下人自己搞不定就抱怨别人产品有bug，你丫的有证据吗？“Show给我看看。”

小张点了几下鼠标，然后轮到我陷入了深思。。。

偶的内心戏：“TMD还真邪门，明明这个CER证书从IIS向导加入就完成了证书安装，而且IIS Manager立即把它显示在了可用证书列表里，怎么退出再进来这证书就消失了？这还真可能是微软的产品质量问题哪。”

内心另一个长了角的声音在叫唤：“作为资深IIS专家和四届微软MVP，你怎么可以陷入深思这么久，笑死！！！别想了，快投降吧，不然午休时间要结束了！！！”

“嗯，这个情况确实蛮奇怪的。。。”我尽量用平静的心情挤出这几句话，然后巧妙的转换了话题，“不过民以食为天啊，不要为了工作废寝忘食嘛。来来来，中午我请你吃个饭吧。换换脑筋，咱们下午一定把它搞定。要是微软产品的bug，找他们技术支持开case可就是免费的啦。”

这小子还蛮听话，拿了钱包和手机就跟出来。挺好，要能找个上菜慢点的馆子吃起来，还能多点时间思考思考。

实话实说，公司楼下这商店今天蛮乱糟糟，满眼都是胸前挂着参观证、背着各式双肩包、拉着各式行李箱的男男女女。如果旁边再多几个漂亮的美女摇摇旗子，那简直是我国壮观旅行团的大聚会啦。不过，他们仅仅是来隔壁的展览中心逛完展览来吃个饭罢了，顺便挤爆下那些看似经济实惠的餐馆。这当然意味着今天我们也免不了在面条馆子门口排个队了。小张正刷着微博，脸上一副放松的神色，完全没有工作没完成泰山压顶的表情，时不时还傻笑下，似乎又遇到某个好笑的段子。

反正这个饭偶是吃得伐开心，倒不是因为此刻被迫排在长长的队伍最后等叫号，而是反省为什么自己没有在之前那几分钟内抓住重点一举攻克那个问题。朦胧地感觉到，一定是某个细节被我简单粗暴地忽略了，而那个细节似乎就隐藏在微软公司某篇文章的角落里。之所以有这种感觉，自然是因为已经和这家公司打过太多的交道，熟到读着某些文字看着某些图片，就差不多知道后面的代码是按照什么思路在写了。那么，这次是哪个细节我没主意哪？“申请证书”-》“安装证书”-》“证书消失”，难道问题出在申请证书这一块？毕竟小张根本没告诉我他从哪里搞了这张证书，即使它在安装后确实出现在了列表里。确定了这个调查方向之后，服务员终于叫了我们的号，两个人才被领进店里。一人叫了一碗牛肉面。

“小张，你买下单，我要去收个快递先。”匆匆吃完面条，我拿上手机直奔办公室而去。

不出所料，小张同学没有锁屏，所以我可以马上看到还留在他电脑桌面的一系列文件，

1. 一张CER证书
1. 一个没有后缀名的文件

似乎是他从CA那边收到的全部文件。这差劲的CA居然连个说明文件也没给。那么这个没有后缀的文件是干什么的？用Notepad++打开一看也是一片乱码，毫无信息量。

也就在这个时候，鼠标乱点到了IIS Manager上的Import…按钮，然后思路豁然开朗！！！

![img-description](/images/iis-server-certs.png)
_图1: IIS 服务器证书管理_

循着这个思路我马上打开了小张机器的控制面板，瞟了眼他刚刚安装过的软件。

![img-description](/images/openssl.png)
_图2: OpenSSL_

“嗯，果然装了这个。”

于是，我开了浏览器瞄了下几个网页，嘴角终于流露出一点点笑容。

“Bing bing bing…”，手机响了，“哦师傅，您在10楼货梯？我马上来我马上来。”随手按了下Win+L，我就向电梯间奔去。

再回到公司的时候，小张已经一脸虔诚的坐在位子上等我了，那小眼神充满了求知的欲望，看得人都不忍心先回座位把快递扔下，只好先走去他那边聊聊。

“老大，吃完饭回来这证书居然自己就好了！！！我真的什么都没动啊！！！好神奇哪！！！”瞧那兴奋劲头，不给他点color see see还真不行。

“哦，那么你下楼之前锁好屏幕了？”我故作轻松的把玩着手里的快递包裹，不去看那个被我一问就由轻松瞬间转为紧张的小眼神。

“老大。。。原来是你。。。”这还差不多，有点悟性。

小张扫了下周围，似乎其他同事还在排队中没回来：“可是问题到底出在哪里啊？老大，你怎么一下子就搞定啦！”

“嗯，让我来简单描述一下之前你的操作步骤，捋一捋，你看看问题出在哪里。”这时候我已经拉过旁边一张空椅子，慢条斯理的进入推理时间。

“首先，你接到任务之后一定是十分仔细的查找了购买证书的流程”，那小子使劲的点了点头，“而且按照我说的找到了一家很便宜的CA。”

“对对。”

“然后你按照他们网站的要求，生成了一个证书请求发送了过去。”

“对啊，对啊。”

“接着他们按照要求给你寄回了证书，你就打开IIS Manager开始尝试安装证书。。。”

“可是，这几步哪里错了哪？”他竟然迫不及待的打断我，伐开心。

“你的证书请求使用OpenSSL制作的吧？”我开始给提示啦。

“是啊，他们网站上面给的指导就是这样的啊，我还特意查过OpenSSL的官方网站，验证了那个步骤是正确的。”小伙子非常有信心的样子。

“那么你有没有查过微软的官方文档，看看IIS的证书应该怎么申请呢？是不是微软也要求你使用OpenSSL呢？”

“这个。。。我想申请证书是个标准流程，用什么工具不都一样吗？”

我笑而不语，直到他安静的等我继续。

“理论上讲，当然使用什么工具效果都是一样的，但是你有看过KB248107, KB295281和KB228821吗？”对面的眼神立即显出几分惊诧。

“看过啊，看过啊，我发现他们都是很老的内容啦，完全和新版IIS对不上号了。”

“No no no”，我摇动下食指，“我提到这几篇KB不是让你照着步骤走，而是你发现了它们隐含了关键信息没有。”

“隐含的关键信息？这里面还有什么信息呢？”

“或者你看过RFC5280没有？”

“这个。。。还要看RFC吗？”

“这个自然要看啊，呵呵。”我很庆幸自己短时记忆力还没怎么衰退，居然能够一口气记下这么多稀奇古怪的编号，“你发送给CA一份请求，他们回寄证书给你。假设这个环节中因为他们的失误该证书不小心发给了另外一个人，那人能不能凭这个证书来伪造我司的官方网站？”

“额。。这个。。应该是不能吧？”

“是啊，不论你采用微软的步骤还是OpenSSL生成证书请求，你的电脑上面一定会保留一点额外的信息，也只有它能保证你拿到证书之后可以正常使用啰。”

“啊，对呀！我懂了我懂了，是那个没有后缀的文件！！！”这小子非常灵光，立即发现了问题的关键，可是立即又一脸困惑：“可是这个文件我完全不知道怎么用啊，IIS Manager只能安装那个CER文件的，我试过的。”

“所以呢，你也试过Import这个对话框？”

![img-description](/images/iis-import.png)
_图3: 导入证书_

“试过啊，当然试过。可是它一定要用PFX文件呢。。。啊？难道我可以制作一个PFX文件？！！”

“当然了，使用OpenSSL你可以把那个没有后缀的文件以及CER文件合并为一个PFX文件。”

“这样啊！”对面开始两眼放光，情绪明显有点小激动，“老大你就是这样把证书装好的啊！”

“嗯嗯。”我觉得是时候回座位了，就拿了包裹起身。在我们唾沫横飞聊的这么一会儿，已经有几个同事拿着外卖回来了。

（完）

B.S.

不出所料，没过几分钟那小子又屁颠屁颠跑到我这边来了：“可是老大，为什么我们之前安装CER文件的时候IIS Manager不抱错呢？而且证书为什么会出现了又消失呢？这难道不是一个问题吗？”

就因为他常常打破沙锅问到底，我非常看好小张未来的发展：“如果是你来设计IIS Manager，你又会怎么处理呢？”

“。。。”

“这确实是一个很难回答的问题。虽然我是IIS的微软MVP，也没办法给出准确答案。”我实话实说，也是希望不要让年轻人抱太大希望，毕竟我们在谈论别人家的产品，既没有源代码也没有人脉上面的联系，“但是，还是可以大胆的猜测一下。”

“啊，猜？”

“那当然啰，代码都是人写的，我们会犯的错误他们也会犯。微软与证书相关的绝大部分文档都是围绕着PFX和CER两个文件后缀的。你刚才是不是已经查过两者的区别了？”

“嗯嗯，是的。CER里面没有包含私钥，而PFX是包含的。那个没有后缀的文件，其实就是我用来申请证书的私钥。”

“那么微软那个对话框为什么只需要你输入CER文件的路径呢？”

![img-description](/images/iis-cert.png)
_图4: 完成证书申请_

“这个。。。”

“或者我换个方式问。IIS Manager上哪里去找私钥呢？”

“喔，它可能是这样的。如果我们通过IIS Manager去申请证书，它会把私钥放在一个安全的地方。等我们收到CER文件，它再把私钥和CER文件合并。就像我们通过OpenSSL手工合并一样。”

我脸上肯定不小心流露出满意的神情，因为那小子脸上居然有一点点得意了。

“所以啊，IIS Manager读取了我们的CER文件，能够获得这张证书里包含的详细信息，自然可以把它加入到列表里面。但是它终归找不到对应的私钥，所以在你刷新列表的时候又把这个证书给扔掉了。”

“是的，老大。”他狡黠的眨了眨眼睛，“我看过MMC的。确实当我们往IIS Manager里面导入CER文件之后那边证书存储里面没有出现这张证书。这说明它出现在IIS Manager列表里面是完全不应该的。”

“对，所以恭喜你了，这确实是微软IIS Manager的一个bug！”

“哈哈哈！”小伙子心满意足的回去他的座位做下一个任务去了。

我一边从抽屉里掏出耳机，一边想着他是不是因为今天这个快乐的经历，就会忘记那碗牛肉面的钱。。。

（全剧终）