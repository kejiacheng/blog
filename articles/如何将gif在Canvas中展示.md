# 如何将gif在Canvas中展示

如何将gif在Canvas中展示，这似乎是一个很简单的问题。相信很多人肯定早以用过Canvas的一个API：drawImage；它的功能就如这个API的名字所示，将图片画入Canvas中，那么我们不妨来试试。

原gif图：

![ke](https://github.com/kejiacheng/blog/blob/master/imgs/%E5%A6%82%E4%BD%95%E5%B0%86gif%E5%9C%A8Canvas%E4%B8%AD%E5%B1%95%E7%A4%BA/1.gif)

drawImage后的图

![ke](https://github.com/kejiacheng/blog/blob/master/imgs/%E5%A6%82%E4%BD%95%E5%B0%86gif%E5%9C%A8Canvas%E4%B8%AD%E5%B1%95%E7%A4%BA/2.png)

从上述的两张图，我们可以看出用了drawImage后，gif只会在Canvas中显示一帧，那么drawImage很明显不是我们要的答案。

## getImageData

getImageData这个API相对drawImage比较冷门，在这里先简单介绍一下：

入参：sx，sy，sw，sh

入参很简单，sx：距左上角的x距离；sy：距左上角的y距离；sw：宽度；sh：高度

出参：imageData对象

imageData对象包括：data，height，width

data：data是一个Uint8ClampedArray（这个类型数组的值是只能为 0-255）数组，其值是以rgba循环组成的（具体的看下面的补充）

width：看MDN上写的是ImageData的实际宽度，但自己试了下都是和入参的sw保持一致，有知道的可以和我说说。

height：看MDN上写的是ImageData的实际高度，但自己试了下都是和入参的sh保持一致，有知道的可以和我说说。

### 补充

不了解的人可能不明白rgba的作用是什么。首先我们要先了解Canvas其实是由一个一个像素组成，比如宽高为400和400的Canvas，那么它就有400 * 400 = 160000个像素，那么这个像素其实就是由rgba组成。那么data里面的值就是一个由左上角循环到右下角的一系列rgba组成，其长度就是：宽度 * 高度 * 4。

## 实现

看了上述imageData的介绍，或许有些人可能已经知道大致如何去实现。其实就是我们获取到gif的每一帧图的data（rgba形成的数组）。然后通过putImageData（这个和getImageData类似，一个是获取，一个是填写），将rgba填入到Canvas中，然后循环每一帧图就实现了我们需要的gif图。

首先我们需要知道每一种文件都是有其特定的格式需要让其他客户端识别的，那么gif图的格式是什么，可以看下面两篇文章。
 [文章地址一（应文）](https://tronche.com/computer-graphics/gif/gif89a.html#image-descriptor),[文章地址二（中文）](http://read.pudn.com/downloads209/doc/984072/GIF%E5%9B%BE%E5%83%8F%E6%A0%BC%E5%BC%8F/GIF%E5%9B%BE%E5%83%8F%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F.pdf)。中文的文章有些地方有些错误，不过大体是没问题。

 那么现在我就依照中文文章大致介绍下gif的格式。

 第一大块是”文件头信息“。其主要是用来识别类型，gif主要包括两种类型：87a和89a。

 第二大块是”屏幕描述块“。其主要是一些逻辑屏幕的信息，比如宽高。

 第三大块是”全局调色板数据“。其内容由rgb循环组成，长度由”屏幕描述块“的GlobalFlag.Palbits决定。

 上述三块是每张gif图头部必定存在的。

 接下来再介绍”图像描述块“、”局部调色板数据“、”扩充块“

 ”图像描述块“，其就是每一帧图像的内容。

 ”局部调色板数据“，该必定在”图像描述块“下方，当”图像描述块“的LocalFlag.LocalPal为1时，则取”局部调色板数据“的数据，否则取”全局调色板数据“的数据。

 ”扩充块“

 “图形控制扩充块”：里面比较重要的一个字段是wDelayTime，其控制着每帧图片间的延迟时间
 “注释说明扩充块”：图形的文字注解说明
 “应用程序扩充块”：里面比较重要的一个字段是Loop Count，其控制着gif的循环次数。当其值为 0 时则代表改gif图无限循环

 “文件结尾块”，gif格的结束块

知道了gif的格式后，我们大致就能根据其获得每帧图片的信息，那么就能将其放置Canvas中实现。

https://github.com/deanm/omggif/blob/master/omggif.js  这个库就是对gif格式的解析，不过自己试了下是有bug的，不建议在项目中使用