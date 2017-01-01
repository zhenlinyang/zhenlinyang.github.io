---
layout: post
title: Unity Android DLL 热更新与加密
date: 2016-12-29 16:00:00.000000000 +08:00
tags: Unity
---

作者：杨振林

## 摘要

本文主要介绍，在安卓环境下，通过更换或解密 Unity-Mono 加载的 Assembly-CSharp.dll 数据文件的方法，达到更新或加密代码的目的。

## 工具与环境

* OS `macOS Sierra 10.12.2`

* Mono `mono unity-5.5` [https://github.com/Unity-Technologies/mono/tree/unity-5.5](https://github.com/Unity-Technologies/mono/tree/unity-5.5)

* Unity `Unity 5.5.0f3 for Mac`

* Java

* Android SDK

* Android NDK `Android NDK r10e`

* Xcode Command Line Tools

* autoconf

* automake

* libtool

* pkg-config

> Xcode Command Line Tools 安装方法 `xcode-select --install`
>
> autoconf automake libtool pkg-config 四项可以使用 Homebrew 安装。
>
> * 获取 Homebrew `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
>
> * 安装 `brew install autoconf automake libtool pkg-config`

## 正文

### 编译 libmono.so

**【步骤一】**选择 Unity 版本对应的 Mono 分支，例如本文选择 Unity 5.5.0f3 则对应 Mono 的版本是 unity-5.5。进入 mono 根目录，mono 根目录是编译 Mono 的工作目录。

**【步骤二】**修改`./external/buildscripts/build_runtime_android.sh`和`./external/buildscripts/build_runtime_android_x86.sh`。

`build_runtime_android_x86.sh`主要负责编译 x86 架构下的 libmono.so。

`build_runtime_android.sh`主要负责编译 arm 架构下的 libmono.so，然后调用`build_runtime_android_x86.sh`。

![](http://source.yangzhenlin.com/unity-android-dll-hotupdate-encrypt/android-x86.png)

![](http://source.yangzhenlin.com/unity-android-dll-hotupdate-encrypt/android.png)

`build_runtime_android_x86.sh`去掉`-g`，去掉调试符号。

`build_runtime_android.sh`将 CFLAGS 下的`-g`修改为`-O2`，去掉调试符号，并增加优化符号。

> `-g` 调试符号
>
> `-O2` 优化符号

注意：“O”是英文第15个字母的大写，不是零。

`build_runtime_android.sh`删除编译`armv5`、`armv6_vfp`，只剩下`armv7a`。

**【步骤三】**在 mono 根目录下，执行`./external/buildscripts/build_runtime_android.sh`。

编译成功后，终端会提示：

```bash
Build SUCCESS!
Build failed? Android STATIC/SHARED library cannot be found... Found        4 libs under builds/embedruntimes/android
total 0
drwxr-xr-x  4 lbs  staff  136 12 30 11:40 armv7a
drwxr-xr-x  4 lbs  staff  136 12 30 11:42 x86
```

在`./builds/embedruntimes/android`目录下，会有`armv7a`和`x86`架构下的`libmono.so`。

### DLL 热更新与加密原理

打开`./mono/metadata/image.c`文件，找到`mono_image_open_from_data_with_name (char *data, guint32 data_len, gboolean need_copy, MonoImageOpenStatus *status, gboolean refonly, const char *name)`函数。

参数说明

`char *data` DLL 数据的指针

`guint32 data_len` DLL 数据的长度

`const char *name` DLL 名

> 使用`glib`库的`g_message`方法，在 Android Logcat 中输出 Assembly-CSharp.dll 的 `name = /data/app/包名-1.apk/assets/bin/Data/Managed/Assembly-CSharp.dll`。

从参数可以看出，进入`mono_image_open_from_data_with_name`函数后，通过辨认`name`，可以对`data`和`data_len`做出修改，从而影响实际加载的 DLL。

### DLL 加密与解密

![](http://source.yangzhenlin.com/unity-android-dll-hotupdate-encrypt/image-c-dll-decryption.png)

从 Unity 导出 Android 工程后，可以拿到`Assembly-CSharp.dll`，在 Unity 中编写的大多数代码都会在这个动态链接库下。通过一些加密算法对此 DLL 进行加密后，初学者使用一些反编译工具就无法看到源代码了。示例工程中，首先将`Assembly-CSharp.dll`的首字节+1（加密），然后在`mono_image_open_from_data_with_name`中将`Assembly-CSharp.dll`的首字节-1（解密），从而实现了对 DLL 的加密过程。

### DLL 热更新

首先在`mono_image_open_from_data_with_name`函数上面补充两个函数。实现从可读写路径中读取 DLL 的操作。

```
static FILE *OpenFileWithPath(const char *path)
{
    const char *fileMode = "rb";
    return fopen (path, fileMode);
}

static char *ReadStringFromFile(const char *pathName, int *size)
{
    FILE *file = OpenFileWithPath (pathName);
    if (file == NULL)
    {
        return 0;
    }
    fseek (file, 0, SEEK_END);
    int length = ftell(file);
    fseek (file, 0, SEEK_SET);
    if (length < 0)
    {
        fclose (file);
        return 0;
    }
    *size = length;
    char *outData = g_try_malloc (length);
    int readLength = fread (outData, 1, length, file);
    fclose(file);
    if (readLength != length)
    {
        g_free (outData);
        return 0;
    }
    return outData;
}
```

改写`mono_image_open_from_data_with_name`函数，当发现可读写路径中存在`/data/data/包名/files/Assembly-CSharp.dll`时，加载新的 DLL。

```
MonoImage *
mono_image_open_from_data_with_name (char *data, guint32 data_len, gboolean need_copy, MonoImageOpenStatus *status, gboolean refonly, const char *name)
{

	////////Modify Begin////////
	int datasize = 0;
	if(name != NULL && strstr (name, "Assembly-CSharp.dll"))
	{
		//重新计算路径，这里假设包名以“com.”开头。
		const char *_pack = strstr (name, "com.");
		const char *_pfie = strstr (name, "-");
		char _name[512];
		memset(_name, 0, 512);
		int _len0 = (int)(_pfie - _pack);
		memcpy(_name, "/data/data/", 11);	//_name = "/data/data/"
		memcpy(_name + 11, _pack, _len0);	//_name = "/data/data/com.Company.ProductName"
		memcpy(_name + 11 + _len0, "/files/Assembly-CSharp.dll", 26);	//_name = "/data/data/com.Company.ProductName/files/Assembly-CSharp.dll"
		char *bytes = ReadStringFromFile (_name, &datasize);
		if (datasize > 0)
		{
			data = bytes;
			data_len = datasize;
		}
	}
	////////Modify End////////

	MonoCLIImageInfo *iinfo;
	MonoImage *image;
	char *datac;

	if (!data || !data_len) {
		if (status)
			*status = MONO_IMAGE_IMAGE_INVALID;
		return NULL;
	}
	datac = data;
	if (need_copy) {
		datac = g_try_malloc (data_len);
		if (!datac) {
			if (status)
				*status = MONO_IMAGE_ERROR_ERRNO;
			return NULL;
		}
		memcpy (datac, data, data_len);
	}

	////////Modify Begin////////
	if(datasize > 0 && data != 0)
	{
		g_free (data);	//释放新 DLL 的数据
	}
	////////Modify End////////

	image = g_new0 (MonoImage, 1);
	image->raw_data = datac;
	image->raw_data_len = data_len;
	image->raw_data_allocated = need_copy;
	image->name = (name == NULL) ? g_strdup_printf ("data-%p", datac) : g_strdup(name);
	iinfo = g_new0 (MonoCLIImageInfo, 1);
	image->image_info = iinfo;
	image->ref_only = refonly;
	image->ref_count = 1;

	image = do_mono_image_load (image, status, TRUE, TRUE);
	if (image == NULL)
		return NULL;

	return register_image (image);
}
```
[热更新 DLL 版本详情 on Github](https://github.com/zhenlinyang/mono/commit/a936f250c33b0fe216f9a3fad37aa3f4c129730e)

### 后续工作

修改代码后，重新编译 Mono。将`./builds/embedruntimes/android`目录中对应`armv7a`和`x86`架构下的`libmono.so`分别替换 Unity 导出的 Android 工程中的`libmono.so`。然后就可以进行安卓打包了。

## 技术支持

[Unity-Technologies/mono](https://github.com/Unity-Technologies/mono)

[Android NDK](https://developer.android.com/ndk/downloads/index.html)

## 参考资料

[Unity3D-重新编译Mono加密DLL](http://www.luzexi.com/unity3d/%E6%B8%B8%E6%88%8F%E6%9E%B6%E6%9E%84/%E5%89%8D%E7%AB%AF%E6%8A%80%E6%9C%AF/2015/04/11/Unity3D-%E9%87%8D%E6%96%B0%E7%BC%96%E8%AF%91Mono%E5%8A%A0%E5%AF%86DLL.html)

[Unity Android 动态更新 Assembly-CSharp.dll](http://blog.sina.com.cn/s/blog_9e5d42ee0102vvtg.html)

[Unity3D研究院之Android加密DLL与破解DLL .SO](http://www.xuanyusong.com/archives/3553)