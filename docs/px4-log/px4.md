# 2023.9.5
## 构造第一个程序px4_simple_app
## qgc操作
- SD卡检查
如果报错
`Crash dumps present on SD,vehicle needs service`
将`COM_ARM_HFLT_CHECK`改为`Disabled`
[点击查看相关博客](https://blog.csdn.net/qq_38768959/article/details/109605241)

![路径选择注意事项](/px4-log/src/1.png)
- px4源码编译用到的pip工具和平常使用的pip工具版本不同，需要用的时候将注释去掉。



