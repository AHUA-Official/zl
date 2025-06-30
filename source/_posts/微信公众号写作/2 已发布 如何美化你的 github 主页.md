## **<font style="color:rgb(89, 89, 89);">  
</font>**
<font style="color:rgb(89, 89, 89);">简单介绍美化 Github 主页的小工具  
</font><font style="color:rgb(89, 89, 89);">关于如何在 github 主页上创建 repo 展示你的主页 见 https://bianyujie.cn/Beautify-your-GitHub-personal-homepage</font>

#### **<font style="color:rgb(0, 0, 0);">贪吃蛇日历</font>**
+ <font style="color:rgb(89, 89, 89);">项目地址： https://github.com/Platane/snk</font>

![](https://cdn.nlark.com/yuque/0/2025/png/39116304/1750068676292-e2a41ff2-9958-4c54-abc9-704129390d51.png)

<font style="color:rgb(136, 136, 136);">image-20241214230254971</font>

<font style="color:rgb(89, 89, 89);">在你的 github 主页提供一个会吃掉你的贡献的贪吃蛇的日历！</font>

+ <font style="color:rgb(89, 89, 89);">原理： 通过 github actions 在你的 codespace 运行代码 每天根据你的提交记录生成对应的 svg 在主页 README.md 应用就好了</font>
+ <font style="color:rgb(89, 89, 89);">使用 看官方文档https://github.com/Platane/snk就OK了 两步走 1 是配置 actions 触发生成 svg 成功， 2 是正确引用 svg 图片</font>
+ <font style="color:rgb(89, 89, 89);">毕竟没有人会拒绝显示一条贪吃蛇吧</font>

#### **<font style="color:rgb(0, 0, 0);">Github Stats</font>**
+ <font style="color:rgb(89, 89, 89);">项目地址： https://github.com/anuraghazra/github-readme-stats</font>

![](https://cdn.nlark.com/yuque/0/2025/png/39116304/1750068676300-1b6ff759-525b-41f9-bfd9-829a04d8d290.png)

<font style="color:rgb(136, 136, 136);">image-20241214231446772</font>

<font style="color:rgb(89, 89, 89);">在你的 github 主页显示提交状态和使用语言卡片！</font>

+ <font style="color:rgb(89, 89, 89);">原理： 通过该团队提供的部署在 Vercel 上的查询 API 返回你的 github 状态卡片</font>
+ <font style="color:rgb(89, 89, 89);">使用 直接使用</font>`<font style="color:rgb(145, 109, 213);">![]()</font>`<font style="color:rgb(89, 89, 89);"> 或者</font>`<font style="color:rgb(145, 109, 213);"><img></font>`<font style="color:rgb(89, 89, 89);">标签即可 参数和主题定制见官方文档即可,列子如下</font>

<font style="color:rgb(89, 89, 89);">https://github.com/anuraghazra/github-readme-stats/blob/master/docs/readme_cn.md</font>

```plain
[![AHUA's GitHub stats](https://github-readme-stats.vercel.app/api?username=AHUA-Official&theme=radical "![AHUA's GitHub stats")](https://github.com/AHUA-Official/github-readme-stats)
```

+ <font style="color:rgb(89, 89, 89);">这个用的人也不少</font>

#### **<font style="color:rgb(0, 0, 0);">个人名片 Cardivo！</font>**
+ <font style="color:rgb(89, 89, 89);">项目地址：https://github.com/satyawikananda/cardivo</font>

![](https://cdn.nlark.com/yuque/0/2025/png/39116304/1750068676449-53c96e16-e10b-4df1-9de3-c1cba5c3c529.png)

<font style="color:rgb(136, 136, 136);">image-20241214232734357</font>

<font style="color:rgb(89, 89, 89);">提供你的个人卡片！支持好看的动画和背景设置，使用 TS 和 Vercel 的 severless 函数服务提供 svg 格式的个人卡片</font>

    - <font style="color:rgb(89, 89, 89);">原理： 同上，vercel 上的 severless 函数</font>
    - <font style="color:rgb(89, 89, 89);">使用： 同上 </font>`<font style="color:rgb(145, 109, 213);">![]()</font>`<font style="color:rgb(89, 89, 89);"> 或者</font>`<font style="color:rgb(145, 109, 213);"><img></font>`<font style="color:rgb(89, 89, 89);">标签 参数见https://github.com/satyawikananda/cardivo</font>

```plain
![Satya wikananda's card name](https://cardivo.vercel.app/api?name=Satya%20Wikananda&description=Hi,%20i%27m%20a%20front%20end%20web%20developer%20and%20i%27m%2020%20y.o.%20Nice%20to%20meet%20you%20%F0%9F%91%8B&image=https://avatars.githubusercontent.com/u/33148052?v=4&backgroundColor=%23ecf0f1&instagram=satyawikananda&linkedin=I%20Gusti%20Ngurah%20Satya%20%20Wikananda&github=satyawikananda&twitter=satya_wikananda&pattern=leaf&colorPattern=%23eaeaea)
```

![]()

<font style="color:rgb(136, 136, 136);">Satya wikananda's card name</font>

+ <font style="color:rgb(89, 89, 89);">这个效果挺好看的 我个人感觉</font>

#### **<font style="color:rgb(0, 0, 0);">萝莉小恶魔</font>**
+ <font style="color:rgb(89, 89, 89);">项目地址 https://github.com/journey-ad/Moe-Counter</font>

![](https://cdn.nlark.com/yuque/0/2025/png/39116304/1750068676327-92d18ec0-9437-4cfa-85e2-f58b1f57bd57.png)

<font style="color:rgb(136, 136, 136);">image-20241214235146665</font>

+ <font style="color:rgb(89, 89, 89);">多种风格可选的萌萌计数器！</font>
+ <font style="color:rgb(89, 89, 89);">很多时候我们想要计数别人看见我们的 website 的次数 使用这个可以使用可爱女孩的风格 LOVE~！</font>
+ <font style="color:rgb(89, 89, 89);">使用： 同上 </font>`<font style="color:rgb(145, 109, 213);">![]()</font>`<font style="color:rgb(89, 89, 89);"> 或者</font>`<font style="color:rgb(145, 109, 213);"><img></font>`<font style="color:rgb(89, 89, 89);">标签 参数见https://count.getloli.com/</font>

#### **<font style="color:rgb(0, 0, 0);">语言图标库</font>**
+ <font style="color:rgb(89, 89, 89);">项目地址 https://github.com/devicons/devicon</font>

![](https://cdn.nlark.com/yuque/0/2025/png/39116304/1750068676613-1c873ce3-368b-4e37-9df8-19c93f3fd0d0.png)

<font style="color:rgb(136, 136, 136);">image-20241215000635299</font>

<font style="color:rgb(89, 89, 89);">如图，你也想通过标签应用展示你接触过的工具吗 使用 devicon 在 jsdelivr 上通过 CDN 分发的语言/工具的 svg 图片！</font>

    - <font style="color:rgb(89, 89, 89);">使用 ： https://devicon.dev/ 项目官网地址，选中你想要展示的工具 如图 左侧会展示 font 和 svg 的引用链接！</font>

```plain
<img src="https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/amazonwebservices/amazonwebservices-plain-wordmark.svg" />
<img src=”https://cdn.jsdelivr.net/gh/devicons/devicon@latest/icons/amazonwebservices/amazonwebservices-plain-wordmark.svg“/>
```

+ ![](https://cdn.nlark.com/yuque/0/2025/png/39116304/1750068676558-9ae68415-45eb-4ec2-a343-852bc7d9ef66.png)

<font style="color:rgb(89, 89, 89);">image-20241215002259898</font>

#### **<font style="color:rgb(0, 0, 0);">Pined</font>**
<font style="color:rgb(89, 89, 89);">使用 Pind 来展示你很好看很用心的开源项目谢谢 如果一个项目的小星星很多 我会很羡慕 很嫉妒</font>

![](https://cdn.nlark.com/yuque/0/2025/png/39116304/1750068676605-ae6bf1ee-127d-4a85-aee5-5138eb69be35.png)

<font style="color:rgb(136, 136, 136);">image-20241215011526095</font>

#### **<font style="color:rgb(0, 0, 0);">好看的主页展示模板</font>**
<font style="color:rgb(89, 89, 89);">https://github.com/kautukkundan/Awesome-Profile-README-templates 提供超级丰富的主页模板的 Readme 类型 不过很遗憾的是 该项目已经蛮久不更新的了 所以我们没有办法抄到最新的模板了</font>

<font style="color:rgb(89, 89, 89);">直接 github 抄你喜欢的人的 readme 使用的工具 我就是这样偷的 一般那些很多粉丝的人我感觉他们的主页都很好看</font>

<font style="color:rgb(89, 89, 89);">如果没有喜欢的人我这里也有简单列举一些</font>

<font style="color:rgb(89, 89, 89);">https://github.com/AHUA-Official 我 抄我谢谢</font>

<font style="color:rgb(89, 89, 89);">https://github.com/Peter-JXL 这个家伙的也好看</font>

<font style="color:rgb(89, 89, 89);">https://github.com/foru17 好看</font>

<font style="color:rgb(89, 89, 89);">https://github.com/DIYgod 好看</font>

<font style="color:rgb(89, 89, 89);">https://github.com/sun0225SUN 好看</font>

<font style="color:rgb(89, 89, 89);">关于更多的我觉得好看的个人主页 看 https://github.com/AHUA-Official/AHUA-Official/blob/main/copyReadmeOffering.md</font>

#### **<font style="color:rgb(0, 0, 0);">工具集合</font>**
<font style="color:rgb(89, 89, 89);">https://github.com/matiassingers/awesome-readme 提供丰富的用于个人主页美化的工具集合</font>

#### **<font style="color:rgb(0, 0, 0);">教程参考</font>**
<font style="color:rgb(89, 89, 89);">https://www.peterjxl.com/Git/GitHub-Profile-Beautify/#%E9%BB%98%E8%AE%A4%E4%B8%BB%E9%A1%B5</font>

<font style="color:rgb(89, 89, 89);">https://bianyujie.cn/Beautify-your-GitHub-personal-homepage</font>

<font style="color:rgb(89, 89, 89);">https://blog.csdn.net/a2360051431/article/details/130945944</font>

<font style="color:rgb(89, 89, 89);">https://zhuanlan.zhihu.com/p/454597068</font>

<font style="color:rgb(89, 89, 89);">嗯 美化主页的方法还有很多 不过我写到这个地方已经想和朋友一起出门吃烧烤烤了 ，那就先这样吧 拜拜！</font>

![](https://cdn.nlark.com/yuque/0/2025/png/39116304/1750068676969-6203ff5a-3dca-4342-9173-427cc3ef894d.png)

<font style="color:rgb(136, 136, 136);">image-20241215011928299</font>

<font style="color:rgb(89, 89, 89);">人生相遇不容易，点个关注我爱你</font>

<font style="color:rgb(89, 89, 89);"></font>

> <font style="color:rgb(89, 89, 89);">来自: </font>[如何美化你的 github 主页](https://mp.weixin.qq.com/s/CnRgD8gBZ6Jp4BR_OAGMpg)
>





