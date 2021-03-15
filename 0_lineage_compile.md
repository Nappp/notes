
最近在自行编译Android系统，便于从底层逆向Android App。本人手上只有一个MacBook，自然希望可以直接编译。可惜事实很残酷，最后只能用Parallels Desktop装了个Ubuntu。把踩过的坑记录下来供后人参考

⚠️ 使用的是LineageOS，而不是更底层的AOSP

⚠️ 下载源码+编译完毕占用至少250G空间，请提前做好准备

### ❌直接下载源码编译

* ##### Mac的文件系统是Case Insensitive的
  
  
### ❌ [hidutil创建dmg文件并挂载](https://stackoverflow.com/questions/8341375/move-android-source-into-case-sensitive-image)

* ##### 到最后`soong`会出现`permission denied`，未找到解决方案
  
  
### ❌ 源码放到硬盘/虚拟机/NAS中，MacOS挂载后编译

* ##### Big Sur系统之后，GateKeeper疯狂拦截make, soong_ui, ninja等命令，失去耐心
  
  
### ✅ Parallels Desktop / VirtualBox(未测试)中编译

* #####	首次编译耗时约20h， 之后每次编译约0.5h ( 2.3Ghz 双核 i5, 16G内存 )



### 简要步骤：

1. 安装Ubuntu
  
2. 安装Python3, pip, [apt-get](https://wiki.lineageos.org/devices/oneplus3/build#install-the-build-packages), 调整ulimit参数到`unlimited`

3. [安装repo命令](https://mirrors.tuna.tsinghua.edu.cn/help/git-repo/)

4. [repo init](https://mirrors.tuna.tsinghua.edu.cn/help/lineageOS/)

注意把其中的`cm-14.1`换成你想要的分支，比如我要Android 10，那就换成`lineage-17.1`

5. `repo sync`，清华镜像容易限流，可以用以下脚本同步到成功为止
  
````
#!/bin/sh
repo sync
while [ $? = 1 ]; do
        echo "======sync failed, re-sync again======"
        sleep 3
        repo sync
done
````
6. `source build/envsetup.sh`

7. `breakfast 机型codename`，可以在[Lineage官网](https://wiki.lineageos.org/devices/)找到

注意这一步可能有很多repo在清华没有，最好在shell设置全局代理

7. 第六步大概率会失败，此时再按照[从rom提取proprietary文件](https://wiki.lineageos.org/extracting_blobs_from_zips.html)的方式就可以从官方获取到proprietary文件，不需要真机

8. 再次`breakfast 机型codename`，这次应该就可以成功了

9. `brunch 机型codename`就可以开始编译了，编译之前建议把以下内容追加到环境变量中
  ````
	export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
	export USE_CCACHE=1
	export CCACHE_EXEC=/usr/bin/ccache
	export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx8192m”
  ````
