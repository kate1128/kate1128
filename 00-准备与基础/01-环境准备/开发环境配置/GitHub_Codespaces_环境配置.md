> 这个可以跳过不配置
>

#  什么是代码空间？
代码空间是托管在云中的开发环境。 可通过将配置文件提交到存储库（通常称为“配置即代码”）来为 GitHub Codespaces 自定义项目，这将为项目的所有用户创建可重复的 codespace 配置。 有关详细信息，请参阅“[开发容器简介](https://docs.github.com/zh/codespaces/setting-up-your-project-for-codespaces/adding-a-dev-container-configuration/introduction-to-dev-containers)”。

[GitHub Codespaces overview - GitHub Docs](https://docs.github.com/en/codespaces/overview)

#  创建第一个 codespace
1. 打开网址链接：[https://github.com/features/codespaces](https://github.com/features/codespaces)
2. 登录你的 GitHub 账户
3. 点击图示 **Your repositories**  
![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735529099446-2a447e97-1616-42be-b665-3acd72dbbd31.png)
4. 进入自己的存储库列表后，点击图示 **New**，新建一个存储库  
![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735529143223-918da836-f523-426f-9406-c146578fd767.png)
5. 这里根据自己需要设置即可，为方便和安全起见 **Add a README file** 建议勾上，同时选择 **Private**（因为课程中用到 API key，注意保护隐私），设置完成后点击 **Create repository**  
![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735529188416-e42ab201-e269-4986-838b-4deed819b612.png)
6. 创建好存储库后，点击 **code** 选择 **Codespaces**, 点击图示 **Create codespace on main**  
![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735529233239-648f8f13-7555-45d8-964b-d6c5a48fc670.png)
7. 等待一段时间后会出现如下界面，接下来操作与 VSCode 相同，可根据需要安装插件调整设置  
![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735529308961-2726294f-4acf-4a76-8e32-39ef3b2392c8.png)

#  环境配置
参照[环境配置](https://www.yuque.com/qiaokate/su87gb/rzhmevuw6xl5sowv)中`1.2 通用环境配置`配置环境即可，可以跳过前两步。

> 由于每个存储库都可以设置一个独立的 codespace，所以这里我们不需要安装 conda 环境。且因为 GitHub 服务器在国外，无需配置国内镜像源。
>

# VSCode 配置 Python 环境
参照[环境配置](https://www.yuque.com/qiaokate/su87gb/rzhmevuw6xl5sowv)中`二、VSCode 配置 Python 环境  
`配置环境即可

> 注意：第一次安装完所有配置后，需要重启一下 codespace
>

# 本地 VSCode 连接 Codespace（非必需）
1. 打开 VSCode，搜索 codespace 安装插件  
![](../figures/C1-6-codespace_plugin.png)![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735529545489-1c2e8b28-1343-47ab-b47c-4c090e67190c.png)
2. 在 VSCode 的活动栏中，单击**远程资源管理器**图标  
![](../figures/C1-6-codespace_connect.png)![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735529601206-c7070f7c-0e68-41c8-8521-aeb8a6517d24.png)
3. 登录 GitHub，根据提示登录即可  
![](../figures/C1-6-GitHub_login.png)![](https://github.com/datawhalechina/llm-universe/raw/main/figures/C1-6-GitHub_login.png)
4. 可以看到这里有我们刚才创建的 codespace，单击红框连接图标  
![](../figures/C1-6-connect_codespace.png)![](https://github.com/datawhalechina/llm-universe/raw/main/figures/C1-6-connect_codespace.png)
5. 成功连接到了 codespace  
![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735529693436-b0a08509-147d-464e-a5be-1275dc99cdc4.png)
6. [VSCode 官方配置文档](https://docs.github.com/en/codespaces/developing-in-a-codespace/using-github-codespaces-in-visual-studio-code)

# 其他
1. 网页关闭后，找到刚才新建的存储库，点击红框框选内容即可重新进入 codespace  
![](../figures/C1-6-restart_codespace.png)![](https://github.com/datawhalechina/llm-universe/raw/main/figures/C1-6-restart_codespace.png)
2. 免费额度  
找到 GitHub 的账户设置后，可以在**Plans and usage**中看到剩余的免费额度  
![](../figures/C1-6-codespace_limit.png)![](https://cdn.nlark.com/yuque/0/2024/png/2639475/1735530079791-97ed984d-3f60-49ed-9900-f3aad061e0b2.png)![](https://github.com/datawhalechina/llm-universe/raw/main/figures/C1-6-codespace_limit.png)
3. codespace 设置，挂起时间建议调整（时间过长会浪费额度）  
![](https://github.com/datawhalechina/llm-universe/raw/main/figures/C1-6-codespace_setting.png)



