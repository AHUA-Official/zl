####  **<font style="color:rgb(0, 0, 0);">摘要</font>**
<font style="color:rgb(89, 89, 89);">讲解了systemctl 命令的用法 并介绍了systemd在linux生命流程中的作用 </font>

#### <font style="color:rgb(0, 0, 0);">参考</font>
鸟哥的Linux私房菜：基础学习篇   第四版    

#### **<font style="color:rgb(0, 0, 0);">Ver 1.0</font>**
#### <font style="color:rgb(0, 0, 0);">动机</font>






#### _<font style="color:rgb(51, 51, 51);">systemctl</font>_<font style="color:rgb(51, 51, 51);"> 最常用的命令行开关。</font>
| <font style="color:rgb(103, 116, 137);">参数</font> | <font style="color:rgb(103, 116, 137);">动作</font> |
| :--- | :--- |
| <font style="color:rgb(51, 51, 51);">-t</font> | <font style="color:rgb(51, 51, 51);">单位类型的逗号分隔值，如服务或套接字</font> |
| <font style="color:rgb(51, 51, 51);">-a</font> | <font style="color:rgb(51, 51, 51);">显示所有加载的单位</font> |
| <font style="color:rgb(51, 51, 51);">--state</font> | <font style="color:rgb(51, 51, 51);">显示处于已定义状态的所有设备：负载，子设备，活动设备，非活动设备等。</font> |
| <font style="color:rgb(51, 51, 51);">-H</font> | <font style="color:rgb(51, 51, 51);">远程执行操作。指定由@分隔的主机名或主机和用户。</font> |


<font style="color:rgb(103, 116, 137);">eg </font>

<font style="color:rgb(103, 116, 137);">查看 运行的所有服务</font>

```plain
systemctl -t service
```

  <font style="color:rgb(6, 6, 7);">列出所有已安装的单元文件</font>

```plain
systemctl list-unit-files | grep enabled
```

#### systemctl 最常用的操作
<font style="color:rgb(51, 51, 51);">可以在服务上执行的主要操作是 -</font>

| start | <font style="color:rgb(103, 116, 137);">开始服务</font> | <font style="color:rgb(103, 116, 137);"></font> |
| :--- | :--- | :--- |
| <font style="color:rgb(51, 51, 51);">stop</font> | <font style="color:rgb(51, 51, 51);">停止服务</font> | <font style="color:rgb(51, 51, 51);"></font> |
| <font style="color:rgb(51, 51, 51);">status</font> | <font style="color:rgb(51, 51, 51);">查看服务状态</font> | <font style="color:rgb(51, 51, 51);"></font> |
| <font style="color:rgb(51, 51, 51);">reload</font> | <font style="color:rgb(51, 51, 51);">重新加载处在非Stopped服务的活动配置</font> | <font style="color:rgb(51, 51, 51);"></font> |
| <font style="color:rgb(51, 51, 51);">restart</font> | <font style="color:rgb(51, 51, 51);">重启服务</font> | <font style="color:rgb(51, 51, 51);"></font> |
| <font style="color:rgb(51, 51, 51);">enable</font> | <font style="color:rgb(51, 51, 51);">开机自启动</font> | <font style="color:rgb(51, 51, 51);"></font> |
| <font style="color:rgb(51, 51, 51);">disable</font> | <font style="color:rgb(51, 51, 51);">不开机自启动</font> | <font style="color:rgb(51, 51, 51);"></font> |
| <font style="color:rgb(51, 51, 51);">is-active</font> | <font style="color:rgb(51, 51, 51);">目前有没有正在运行中</font> | <font style="color:rgb(51, 51, 51);"></font> |
| <font style="color:rgb(51, 51, 51);">is-enable</font> | <font style="color:rgb(51, 51, 51);">开机时有没有默认要启用这个 unit</font> | <font style="color:rgb(51, 51, 51);"></font> |




