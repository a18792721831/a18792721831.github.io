---
layout: post
title: AnimateDiffusion文字生成图片--入门
categories: [ai, sd]
description: AnimateDiffusion文字生成图片--入门
keywords: sd, sd webui, Stable Diffusion, 文字生成图片, ai
---

AnimateDiffusion文字生成图片--入门

> 下面基本上所有的操作都需要访问外网，请自行解决外网。

# 1. 安装
首先使用`git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git` 下载代码。  

![img.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img.png)

然后使用`conda create -n aidraw python=3.10` 创建一个python环境，如果没有conda,那么可以使用如下命令安装conda:
```Shell
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm -rf ~/miniconda3/miniconda.sh
```
安装完成后使用`conda init bash`初始化conda，然后就可以使用conda了。
> 这里有个需要注意的点，仓库里面说是python的版本是3.10.6，这里有点问题，因为目前python中3.10大版本中最新的已经是3.10.13了，如果你还是用3.10.6，那么安装的依赖会让你升级python才能用的问题。所以我们只需要指定大版本为3.10就行了，第三位的小版本直接使用最新的就行。

![img_1.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_1.png)


然后就可以使用提示的命令激活和退出环境了：  
![img_2.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_2.png)



![img_3.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_3.png)


然后使用如下命令启动:
```Shell
python3 launch.py --autolaunch --listen --xformers --force-enable-xformers --precision full --no-half --enable-insecure-extension-access
```
其中的选项，`autolaunch`表示自动安装依赖，`listen`表示监听的地址为`0.0.0.0`否则会监听`127.0.0.1`只能本机访问，如果你是在远程linux上运行，那么这个最好设置下。
`xformers`表示使用`xformers`加速推理，`force-enable-xformers`表示强制开启，`enable-insecure-extension-access`表示允许插件。

> 更多信息参考： https://github.com/sudoskys/StableDiffusionBook/blob/main/docs/install/WebUi/launch.md

运行之后就会开始下载依赖等等：  
![img_4.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_4.png)



当运行成功后，就可以将上面命令中的`launch.py`换成`webui.py`，因为命令比较长，可以使用`echo 'alias aiweb_start="python3 webui.py --autolaunch --listen --xformers --force-enable-xformers --precision full --no-half --enable-insecure-extension-access"' >> ~/.bash_profile`
将这个巨长的启动命令设置为别名，设置后使用`source ~/.bash_profile`生效。  
然后就会出现这个错误：  
![img_5.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_5.png)


这表示缺少一个依赖，使用`pip install opencv-python-headless`下载缺少的依赖：  
![img_6.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_6.png)


下载成功后重新启动(可以使用`launch.py`，也可以使用`webui.py`启动)就会下载一个基础的模型：  
![img_7.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_7.png)


耐心等待下载完成就行  
![img_8.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_8.png)


根据日志查看就会发现已经成功启动了，监听的端口是7860，然后就可以访问了：  
![img_9.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_9.png)


访问：  
![img_10.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_10.png)


然后我们小小的试一把：  
![img_11.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_11.png)


然后点击`enerate`生成：  
![img_12.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_12.png)


如果你的环境没有GPU，可以去掉启动命令中的`--xformers --force-enable-xformers`，然后启动，表示使用CPU进行计算。我自己的环境，使用CPU生成相同的提示词的图片，需要大概80~120秒，使用GPU大概15秒左右，差距还是挺大的。  
![img_13.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_13.png)


可以使用`batch count`一次生成多张图片，下面的`batch size`表示执行几次，两者相乘，就是最终生成的图片数量，需要注意，生成的图片越多，那么耗费的时间越长。  
![img_14.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_14.png)

当然你也可以使用中文的提示词。  
比如加个`性感`，然后生成：  
![img_15.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_15.png)

如果出现内存，或者显卡的显存不足，尝试使用`--medvram`或者`--lowvram`提示使用低内存，但是可能需要的时间会更长。  
如果对于大图，觉得细节比较粗糙，可以开启精细化：  
![img_16.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_16.png)


而且可以限定生成的人物数量,以及开启随机种子：  
![img_17.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_17.png)


一般在开启精细化处理的时候，会出现内存不足等问题。精细化相当于在整体图片不变的基础上，将图片分割为一个个小块，然后针对小块重新用大图的方式绘制，绘制完成后再合并。  
![img_18.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_18.png)


所以可以看到比之前需要计算的迭代次数也增加了，而且数据量也大了。  
![img_19.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_19.png)


结果确实高清了不少

![img_20.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_20.png)


# 2. 插件
## 2.1 汉化插件
英文的界面，没有中文，全靠猜。  
在启动的时候加了允许插件的参数，所以可以安装插件：  
![img_21.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_21.png)


这里表示从`https://raw.githubusercontent.com/AUTOMATIC1111/stable-diffusion-webui-extensions/master/index.json` 加载插件列表。  
![img_22.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_22.png)


不过这里面没有汉化包，所以使用指定url下载插件`https://github.com/VinsonLaro/stable-diffusion-webui-chinese`:  
![img_23.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_23.png)


![img_24.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_24.png)


下载完成后需要重启生效：  
![img_25.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_25.png)


然后就会重启：  
![img_26.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_26.png)


重启后只是加载了插件，还没有使用呢：

![img_27.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_27.png)

![img_28.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_28.png)



![img_29.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_29.png)






然后就汉化了：  
![img_30.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_30.png)


## 2.2 中文提示词插件
在插件列表中这个插件可以安装下，毕竟中文提示词的效果其实比较差，所以有了中文提示词转为英文提示词的插件：  
![img_31.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_31.png)


点击安装就行了，需要注意的是，每次安装了插件，都需要重启哦。  
![img_32.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_32.png)


这就是刚安装的插件，试试：  
![img_33.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_33.png)


效果真不错，你就不用自己写了，点点点，你心中的她就出来了[偷笑]
还可以选择负面词，需要注意的是，`右键`是加入到负面提示词里面的哦，这个牢记：  
![img_34.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_34.png)


最后看看效果：  
![img_35.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_35.png)


## 2.3 模型下载插件
增加一个插件`https://github.com/tzwm/sd-webui-model-downloader-cn` 可以在webui中下载模型，而不需要手动下载。  
![img_36.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_36.png)


出现更新失败，可能是因为我们没有配置Civitai的秘钥，不管它，先下载一个试试：  
以 `https://civitai.com/models/228525/ultra` 为例：  
将上面的地址输入到插件中，需要去掉最后一层哈，然后点击预览    
![img_37.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_37.png)


然后点击下载就行了，因为模型毕竟在国外，所以预览可能失败，多点击几次。  
![img_38.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_38.png)


等待下载成功就行了。  
![img_39.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_39.png)


下载成功就可以刷新并选择新的模型了，试试：  
下载必须有秘钥，否则会失败。。  
![img_40.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_40.png)



## 2.4 模型下载
另一种方式是使用命令行手动下载，这样可以不需要秘钥：  
首先在`~/.bash_profile`中加入如下内容：
```Shell
function download_model {   
        wget "https://g.blfrp.cn/civitai.com/api/download/models/$1" --content-disposition
}
```
然后`source ~/.bash_profile`生效  
然后切换到`stable-diffusion-webui/models/Stable-diffusion`目录，使用`download_model 228525`下载：  
这种方式下载存在一定的滞后性，可能模型还没同步到加速站，出现404问题。  
我下载使用的方式：  
![img_41.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_41.png)

这里的id值是从c站拿到的：  
![img_42.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_42.png)


先选择版本，然后使用版本号就可以下载了。  
这种方式相对成功率高一点。
## 2.5 c站helper插件
如果上面两种方式都不好使，还有一个插件：`https://github.com/butaixianran/Stable-Diffusion-Webui-Civitai-Helper`  
这个插件也是下载模型的，具体哪个能用，哪个好使，需要自己选择。(我用的是这个)  
安装插件都是一样的。  

![img_43.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_43.png)

安装好后，将模型地址填入，获取模型信息，这里也多试几次，网络问题：  
![img_44.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_44.png)


记得选择路径和版本：  
![img_45.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_45.png)


然后点击下载就行了.
下载模型如果没有秘钥，是一个麻烦的事情。
我下载使用的方式：  
![img_46.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_46.png)


这里的id值是从c站拿到的：  
![img_47.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_47.png)


先选择版本，然后使用版本号就可以下载了。

## 2.6 c站秘钥
![img_48.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_48.png)



# 3. 模型
上面我们玩的只是其中的一个模型，实际上训练的数据不同，生成的效果也不同。  
可以到`https://civitai.com/` 查看更多的模型，以及效果比较好的case.  
比如： `https://civitai.com/models/18523/magmix` 或者 `https://civitai.com/models/228525/ultra`  
![img_49.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_49.png)


模型下载后就可以选择使用了：  
![img_50.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_50.png)


使用提示词，生成
![img_51.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_51.png)


提示词：
```text
正向：
front view,full_body,best quality,Highly detailed,realistic,full body,1girl,solo,female,young,big breasts,unbuttoned clothes,gym leader,oval face,Cupid mouth,slender,glamor,shiny skin,bare_legs,swimsuit,long hair,ponytail,bangs,hair behind ear,High leg lift,Hands on waist,v,high kick,wide eyed,bright pupils,Slightly open mouth,seductive smile,bikini,necklace,earrings,bracelet,summer,grasslands,
反向：
multiple breasts, (mutated hands and fingers:1.5 ), (long body :1.3), (mutation, poorly drawn :1.2) , black-white, bad anatomy, liquid body, liquid tongue, disfigured, malformed, mutated, anatomical nonsense, text font ui, error, malformed hands, long neck, blurred, lowers, lowres, bad anatomy, bad proportions, bad shadow, uncoordinated body, unnatural body, fused breasts, bad breasts, huge breasts, poorly drawn breasts, extra breasts, liquid breasts, heavy breasts, missing breasts, huge haunch, huge thighs, huge calf, bad hands, fused hand, missing hand, disappearing arms, disappearing thigh, disappearing calf, disappearing legs, fused ears, bad ears, poorly drawn ears, extra ears, liquid ears, heavy ears, missing ears, fused animal ears, bad animal ears, poorly drawn animal ears, extra animal ears, liquid animal ears, heavy animal ears, missing animal ears, text, ui, error, missing fingers, missing limb, fused fingers, one hand with more than 5 fingers, one hand with less than 5 fingers, one hand with more than 5 digit, one hand with less than 5 digit, extra digit, fewer digits, fused digit, missing digit, bad digit, liquid digit, colorful tongue, black tongue, cropped, watermark, username, blurry, JPEG artifacts, signature, 3D, 3D game, 3D game scene, 3D character, malformed feet, extra feet, bad feet, poorly drawn feet, fused feet, missing feet, extra shoes, bad shoes, fused shoes, more than two shoes, poorly drawn shoes, bad gloves, poorly drawn gloves, fused gloves, bad cum, poorly drawn cum, fused cum, bad hairs, poorly drawn hairs, fused hairs, big muscles, ugly, bad face, fused face, poorly drawn face, cloned face, big face, long face, bad eyes, fused eyes poorly drawn eyes, extra eyes, malformed limbs, more than 2 nipples, missing nipples, different nipples, fused nipples, bad nipples, poorly drawn nipples, black nipples, colorful nipples, gross proportions. short arm, (((missing arms))), missing thighs, missing calf, missing legs, mutation, duplicate, morbid, mutilated, poorly drawn hands, more than 1 left hand, more than 1 right hand, deformed, (blurry), disfigured, missing legs, extra arms, extra thighs, more than 2 thighs, extra calf, fused calf, extra legs, bad knee, extra knee, more than 2 legs, bad tails, bad mouth, fused mouth, poorly drawn mouth, bad tongue, tongue within mouth, too long tongue, black tongue, big mouth, cracked mouth, bad mouth, dirty face, dirty teeth, dirty pantie, fused pantie, poorly drawn pantie, fused cloth, poorly drawn cloth, bad pantie, yellow teeth, thick lips, bad cameltoe, colorful cameltoe, bad asshole, poorly drawn asshole, fused asshole, missing asshole, bad anus, bad pussy, bad crotch, bad crotch seam, fused anus, fused pussy, fused anus, fused crotch, poorly drawn crotch, fused seam, poorly drawn anus, poorly drawn pussy, poorly drawn crotch, poorly drawn crotch seam, bad thigh gap, missing thigh gap, fused thigh gap, liquid thigh gap, poorly drawn thigh gap, poorly drawn anus, bad collarbone, fused collarbone, missing collarbone, liquid collarbone, strong girl, obesity, worst quality, low quality, normal quality, liquid tentacles, bad tentacles, poorly drawn tentacles, split tentacles, fused tentacles, missing clit, bad clit, fused clit, colorful clit, black clit, liquid clit, QR code, bar code, censored, safety panties, safety knickers, beard, furry ,pony, pubic hair, mosaic, excrement, faeces, shit, futa, testis,scowl,disdain,
```
调整下高清，给与更多的引导：  
![img_52.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_52.png)

![img_3.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_53.png)


使用了细化器，导致风格变了，还是关闭细化器吧  

![img_2.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_54.png)
看看另一个模型：
![img_1.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_55.png)

这个还不错  
![img.png](/images/posts/2024-02-06-AnimateDiffusion文字生成图片--入门/img_56.png)



# 4. 总结
这次只是使用了一个文本到图片的功能，实际上`AnimateDiffusion`还是非常强大的，继续探索。

> 最后祝愿各位玩的开心！~
