# fiddler抓包

> 在互联网时代数据的重要性不言而喻，各大平台都在进行数据的采集分析，大部分接触的都是电脑和服务器上面布置程序进行爬虫采集，而随着移动端用的越来越广泛，手机爬虫也开始被人用起，今天分享一个雷电模拟器爬虫设置。

操作步骤如下:

1. 下载最新版Fiddler,强烈建议在官网下载：**https://www.telerik.com/download/fiddler**

<img src="attachments/image-20230308140419376.png" alt="image-20230308140419376" style="zoom:50%;" />

2. 正常傻瓜式安装，下一步，下一步，安装完毕后，先不用急于打开软件
3. 下载并安装Fiddler证书生成器：**http://www.telerik.com/docs/default-source/fiddler/addons/fiddlercertmaker.exe?sfvrsn=2**

- 打开Fiddler，点击工具栏中的Tools—>Options

<img src="attachments/v2-5424e8310bb32eb7eaf6b81c6eceea52_r.jpg" alt="img" style="zoom:50%;" />

- 点击HTTPS，勾选Decrypt HTTPS traffic和Ignore server certificate(unsafe)

<img src="attachments/v2-94f138bcdfb758a70be47d84703bffe1_r.jpg" alt="img" style="zoom:50%;" />

- 点击Actions，点击Export Root Certificate to Desktop

<img src="attachments/v2-f2d57cacfb47045d27d6e249979a64cc_r.jpg" alt="img" style="zoom:50%;" />

【注】此时电脑上会生成 一个证书

![img](attachments/v2-e91b7344acfe3439fd75b044d73407b1_r.jpg )

- https设置及connections设置，勾选选择项

<img src="attachments/v2-748a74e11a5a7d7c151e4d07561a1edf_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-9cf9236ed3d939d103fb5061426da7c0_r.jpg" alt="img" style="zoom:50%;" />

- 安装雷电模拟器[下载地址](https://http://www.ldmnq.com/)

<img src="attachments/image-20230308142758065.png" alt="image-20230308142758065" style="zoom:50%;" />

- 安装好后，桌面双击打开雷电模拟器，点击设置

<img src="attachments/v2-7914c3d9133e22d7738f46b8972dc797_r.jpg" alt="img" style="zoom:50%;" />

- 安装好后，桌面双击打开雷电模拟器，点击设置

<img src="attachments/v2-38bc473c57d8befda99d9723f1d261fa_r-1678256932588-19.jpg" alt="img" style="zoom:50%;" />

- 选择网络设置，勾选桥接模式，点击安装驱动，点击确定，点击保存设置

<img src="attachments/v2-e0af8604f27ffc73c3c3ac62bb5d8d70_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-280d651ebf4dad77b3ca8471bc4cb92c_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-439bdba48dbc2b38083262993c12e635_r.jpg" alt="img" style="zoom:50%;" />

- 打开模拟器，设置代理。找到系统应用，点击设置，点击无线网络WLAN—>左键常按点击已连接网络—>修改网络 。

<img src="attachments/v2-0842b99b4384d0bf8270955ae2c6f33d_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-34de479f0a8a1644aefc5ef8221ef985_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-117d8645ad1d07ebeaad191d3d86c355_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-4459a6c1e858489a1c25e3a38e5ebee4_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-bb1125afb1c0244f1de077f860eae02a_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-ce56261ab2fa4d4877699c87dbd2eb65_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-ca13724cc8e58af3ce290f013a39c3a5_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-5db7a292774712d8bdaeea35790cfb9b_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-86673c8b3addcb80a7af2b3a992dcfe2_r.jpg" alt="img" style="zoom:50%;" />

- 将步骤6导出的证书FiddlerRoot.cer文件导入至模拟器

<img src="attachments/v2-cdee1590344bc168e4497fdac6797a67_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-58efc37f691495179a1d486eef025291_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-55c1eab7e94e728cf617ed14cccaa23c_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-16c685f5209871f80969304ed2966460_720w.webp" alt="img" style="zoom:50%;" />

<img src="attachments/v2-49e361aff505b10c0924287e2e55c1e1_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-0ba4b123d92b5e27e1ca7cba88b081f0_r.jpg" alt="img" style="zoom:50%;" />

<img src="attachments/v2-b0f4828f0d4f802f8884f2360d5ab1cd_720w.webp" alt="img" style="zoom:50%;" />

<img src="attachments/v2-b0f4828f0d4f802f8884f2360d5ab1cd_720w-1678257103887-60.webp" alt="img" style="zoom:50%;" />

<img src="attachments/v2-84170100fa7d1229c33a397b640c1d52_720w.webp" alt="img" style="zoom:50%;" />

- 在模拟器中打开系统应用—>设置—>安全—>从SD卡安装。找到FiddlerRoot.cer文件，按提示导入即可，注意在此过程需要名称和解锁图案等，自行即可

<img src="attachments/v2-6e86f17f2d0915e437d54cc1bc31ab4c_720w.webp" alt="img" style="zoom:50%;" />

<img src="attachments/v2-60182722fb168b9075d60d77bbce91b7_720w.webp" alt="img" style="zoom:50%;" />

<img src="attachments/v2-984ad50ca4b2b97f0f1df5d0507bbd64_720w.webp" alt="img" style="zoom:50%;" />

<img src="attachments/v2-a5ecf09cec6cdfc957b126f761474fc3_720w.webp" alt="img" style="zoom:50%;" />

<img src="attachments/v2-4e34bfd2c8e2fbdea999e597584a95c8_720w.webp" alt="img" style="zoom:50%;" />

<img src="attachments/v2-98cb971a8de05194bb18941d36f035f6_720w.webp" alt="img" style="zoom:50%;" />

<img src="attachments/v2-8277201b672ad1dfda4df22cdd9a3500_720w.webp" alt="img" style="zoom:50%;" />

- 打开fiddler，重启模拟器，输入设置的密码，按回车键，打开需要抓包的APP，就可以在电脑上进行APP抓包了

<img src="attachments/v2-a352f64f845ef5312f5f41235501fe3d_720w.webp" alt="img" style="zoom:50%;" />