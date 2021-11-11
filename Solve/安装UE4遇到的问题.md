- Setup.bat执行完之后，执行窗口没了
  - 正常现象
- GenerateProjectFiles.bat执行完之后：error MSB3644：未找到框架“.NetFrame...
  - 解决方法：运行Visual Studio Installer，点击单个组件，勾选.Net Framework 4.6.2 SDK和.Net Framework 4.6.2 目标包，然后点击修改
- 运行UE4Editor.exe，出现损坏的映像弹框
  - 解决方法：删除UnrealEngine\Engine\Binaries\Win64下的内容，然后生成解决方案。或者直接重新生成。
- D8049：无法执行.... 命令行太长 无法适应调试记录
  - 解决方法：修改安装路径的长度
  - https://www.bilibili.com/read/cv10094364/（目前）
  - https://blog.csdn.net/qq_42855293/article/details/109716507
- C3859: 超过了 PCH 的虚拟内存范围
- C1076：编译器限制：达到内部堆限制

- C1002: 在第 2 遍中编译器的堆空间不足





- https://blog.csdn.net/weixin_43704737/article/details/106307112
- https://blog.csdn.net/tianttt/article/details/42295899