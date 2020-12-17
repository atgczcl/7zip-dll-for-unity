# 7zip-dll-for-unity
7zip-dll for unity

感谢作者：YzlCoder！

我合并成了一个工程，编译为一个7Zipdll.dll了！共享！
使用说明：
尊重原创，转载请注明出处，谢谢！

http://blog.csdn.net/y1196645376/article/details/52492294



我们知道在Unity5.x开始推出新的AssetBundle打包策略，给出的新的打包以及解包的Api。在处理资源依赖关系的时候较旧版本的显得方便了许多。



然而，在使用的过程中我们发现，新的解包Api( AssetBundle.CreateFromFile )不支持压缩过的AssetBundle。也就是说要求在打包AssetBundle的时候要求使用 BuildAssetBundleOptions.UncompressedAssetBundle模式。所以，我们就要自己对打包好的资源进行压缩，解压。


这里我主要介绍两个压缩方式：LZMA ( Unity对于AssetBundle原本的压缩方式 ) ,GZIP

这两个压缩库在官网上都是能够下到源码。不过，源码中给出的压缩函数都是同步的。所以，不能够直接在Unity中使用。而且，压缩过程中不支持返回progress(进度)。所以我修改了源码一些地方，对压缩、解压文件的接口封装了一下，然后添加了progress回调接口。至于修改地方我就不大半页去叙述，而且也没必要叙述。我会给出两个工具源码的下载地址和我修改后的源码地址。如果有兴趣的可以自行阅读修改。这篇文章主要是讲这个工具如何使用。


LZMA 官网下载地址：http://www.7-zip.org/sdk.html

GZIP 官网下载地址 ：http://icsharpcode.github.io/SharpZipLib/

我修改后的工程(包含LZMA,GZIP,UPK)已经推送到Git上了：http://git.oschina.net/Wahh_1314/Compress

值得注意的是：由于GZIP的源码中包含了C#6.0的语法，可能导致VS2013无法正确编译该工程。所以大家如果只是想使用可以直接使用我修改后工程中已经生成好的Dll文件。如果想要修改代码重新生成请使用VS2015。

下面我主要是给出一个例子讲解如何使用这两个压缩库：


1.首先把我修改后的工程下载下来，从里面的Debug文件中找到这几个Dll文件：


2.创建一个Unity新的工程.在工程目录里面创建一个名为Plugins的文件夹。然后将这四个Dll文件导入到这个文件夹中。



3.然后随便导入一个稍微比较大的一个资源文件,用于测试压缩。比如我这里导入了一个音频文件。




4.然后开始写代码了。创建一个Test.cs，然后代码如下：


using UnityEngine;
using System.Collections;
using YZL.Compress.GZip;
using YZL.Compress.LZMA;
public class Test : MonoBehaviour {
 
 
    /// <summary>
    /// 请使用异步压缩或者解压。
    /// </summary>
    void OnGUI()
    {
        if (GUI.Button(new Rect(Screen.width/2 - 210 , 200, 140, 80), "LMZA压缩文件"))
        {
            LZMAFile.CompressAsync(Application.dataPath + "/music.mp3", Application.dataPath + "/music.lzma", null);
        }
 
        if (GUI.Button(new Rect(Screen.width / 2 + 70, 200, 140, 80), "GZIP压缩文件"))
        {
            GZipFile.CompressAsync(Application.dataPath + "/music.mp3", Application.dataPath + "/music.gzip", null);
        }
        if (GUI.Button(new Rect(Screen.width / 2 - 210, 320, 140, 80), "LMZA解压文件"))
        {
            LZMAFile.DeCompressAsync(Application.dataPath + "/music.lzma", Application.dataPath + "/lzmamusic.mp3", null);
        }
        if (GUI.Button(new Rect(Screen.width / 2 + 70, 320, 140, 80), "GZIP解压文件"))
        {
            GZipFile.DeCompressAsync(Application.dataPath + "/music.gzip", Application.dataPath + "/gzipmusic.mp3", null);
        }
    }
}

值得注意：如果你要使用上面的代码，你要保证你的工程目录中也有跟我一样位置的一个music.mp3文件。函数调用都是传入两个地址和一个回调委托。前两个参数好理解，就是源文件路径和经过操作后的存储路径。第三个参数是进度回调委托。类型为 public delegate void ProgressDelegate(long fileSize, long processSize);
fileSize：表示文件总大小,processSize：表示进度当前大小。processSize / fileSize 就表示整个操作的进度了。

支持进度回调就可以干很多事情了，比如：在压缩的时候可以显示进度条等等。



5.比如我写下这样一个函数,在上面的代码中当做第三个参数传入进去：


void ShowProgress(long all,long now)
{
     double progress = (double)now /all;
     Debug.Log("当前进度为: " + progress);
}

6.运行,点击LMZA压缩Button，Console就会出现压缩进度的日志信息。当压缩完毕之后，当前目录就会生成music.lzma这个已经压缩好的文件了。





7.重新运行，点击LMZA解压Button，就可以将刚才压缩的文件重新解压出来。



值得注意的事情是：因为有两个压缩方式GZIP和LZMA，在实际工程中只会选其一。

那么就根据需要选择：

LZMA压缩方式：Compress.Info.dll和Compress.LZMA.dll 文件   

或者  

GZip压缩方式：Compress.Info.dll和ICSharpCode.SharpZipLib.dll 文件



也不要将两个压缩方式压缩的文件混用，也就是说其一压缩后的文件，请不要用另一种方式尝试去解压！！



以上内容基本上介绍完了压缩库怎么使用，下面介绍一下文件夹打包工具。



实际项目中，打包好了AssetBundle，但是一个文件夹的所有AssetBundle不可能每个单独去压缩，这样解压的时候也很麻烦。所有我们可以先将这个文件夹先打包，然后直接对这个文件夹压缩即可。

打包需要的库文件：Compress.UPK.dll    


1.还是在刚才的工程导入一个文件夹，我在里面放了几张图片。





2.继续修改Test.cs，在OnGUI函数中继续添加Button,刚才写的四个Button请不要删除。

if (GUI.Button(new Rect(Screen.width / 2 - 210, 440, 140, 80), "UPK打包文件夹"))
{
    UPKFolder.PackFolderAsync(Application.dataPath + "/picture", Application.dataPath + "/picture.upk", ShowProgress);
}
if (GUI.Button(new Rect(Screen.width / 2 + 70, 440, 140, 80), "UPK解包文件夹"))
{
    UPKFolder.UnPackFolderAsync(Application.dataPath + "/picture.upk", Application.dataPath + "/", ShowProgress);
}


3.下面来测试一下打包工具，运行，点击UPK打包：



已经生成了upk文件。





4.这个时候先把picture文件夹删除，这样就只有upk了。重新运行，点击UPK解包。





发现之前的picture文件夹已经被解包出来了。



Unity测试工程：https://git.oschina.net/Wahh_1314/CompressDemo



OK介绍基本上就是这样了，如果有不懂的地方请在下面评价回复。如果发现代码有错的地方，请在下面评论指出。谢谢。



