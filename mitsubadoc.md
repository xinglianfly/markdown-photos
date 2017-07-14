 # mitsuba 读取xml
mitsuba 使用xerces-c++类库来解析xml，这个类库可以很方便的读写xml数据。使用DOM，SAX SAX2 APIS来解析、生成、操作、验证xml。官网在[这里](https://xerces.apache.org/xerces-c/)。如果你想看他的使用方法的话，可以看[programing guide](https://xerces.apache.org/xerces-c/program-3.html)。在mitsuba中，主要使用了xerces-c++中的SAX API来解析xml文件。(建议自己写一个demo感受一下这个类库)
 ## xerces-c++类库
xerces-c++中的SAX 是基于事件的xml的解析器。与DOM比较而言，SAX是一种轻量型的方法。我们知道，在处理DOM的时候，我们需要读入整个的XML文档，然后在内存中创建DOM树，生成DOM树上的每个Node对象。当文档比较小的时候，这不会造成什么问题，但是一旦文档大起来，处理DOM就会变得相当费时费力。特别是其对于内存的需求，也将是成倍的增长，以至于在某些应用中使用DOM是一件很不划算的事（比如在applet中）。这时候，一个较好的替代解决方法就是SAX。
SAX在概念上与DOM完全不同。它不同于DOM的文档驱动，它是事件驱动的，它并不需要读入整个文档，而文档的读入过程也就是SAX的解析过程。（边读边解析）所谓事件驱动，是指一种基于回调（callback）机制的程序运行方法。
 ## 快速上手
下面这个图是mitsuba的运行界面，箭头指的那个是mitsuba的log输出。
![界面](https://github.com/xinglianfly/markdown-photos/blob/master/mitsubaxml/addd795b-2145-483f-93d1-332f2db8bb23.png?raw=true)

下面这个图的框里可以看到这个地方是xml解析的开始部分，先进行加载xml的操作。在代码中找到这句话是在sceneloader.cpp里面
![log](https://github.com/xinglianfly/markdown-photos/blob/master/mitsubaxml/175bcf6d-d836-4b8f-bed2-bb069d4c3aa6.png?raw=true)
 
通过研究代码可以发现，xerces解析xml的逻辑都放在了scenehandler.cpp里面
 ## SceneHandler.cpp
这个handler是mitsuba解析xml的主要逻辑。使用xerces-c++这个类库的时候，需要重写几个方法，分别是```void endElement(const XMLCh *,const xmlName)```、```void
startElement(const XMLCh* const xmlName,AttributeList &xmlAttributes)``` 等方法。这两个方法可以检测到xml是以什么标签结尾和以什么标签开始。这两个方法里面有switch，当读到某个标签时对应相应的解析操作。
 下面是xml中所有的标签，可以看到，mitsuba将其中的标签直接和相对应的类关联。
![](https://github.com/xinglianfly/markdown-photos/blob/master/mitsubaxml/8bbd3fdc-0dff-47de-af8f-5f0e1a3b0cb2.png?raw=true)

 mitsuba创建了一个上下文对象```
    ParseContext &context = m_context.top();```，当读到某个标签的属性的时候，可以把属性存到context里面，这个context最后会传给一个Scene对象，用来构建场景。

## xml场景文件
![](https://github.com/xinglianfly/markdown-photos/blob/master/mitsubaxml/bc852441-09f2-416b-97e5-22a163e20dfd.png?raw=true)

上面这个图片是mitsuba场景文件的一部分，可以看到里面都是由标签构成的，这些标签都有相应的属性，当类库读到这些属性的时候，会做相应的操作，比如说，如果遇到了**type**这个属性，那么mitsuba就会加载相应的插件，插件的名字就是**type**的值。还有一些标签有**name**和**value**属性，这些标签会被存到context的属性里面。
当场景的属性传到scene中的时候，开始构建sah tree。并且在scene这个类中做求交等操作。
 
