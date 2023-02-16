<!--
 * @Author: Donald duck tang5722917@163.com
 * @Date: 2023-02-16 20:10:30
 * @LastEditors: Donald duck tang5722917@163.com
 * @LastEditTime: 2023-02-16 20:11:06
 * @FilePath: \mysticism-mud-doc\Mud教程\Mudcore登录启动流程.md
 * @Description: 
 * 
 * Copyright (c) 2023 by Donald duck tang5722917@163.com, All Rights Reserved. 
-->
刚完成了LPMUD启动过程的学习<https://bbs.mud.ren/threads/8>  
我做了一下Mudcore的启动启动流程梳理  
1. 载入simul_efun
2. 运行master.c文件
3. 如果玩家连线，自动调用master中的connect 方法，并返回连线对象login_ob  
   其中login_ob通过:  
`err = catch (login_ob = new(LOGIN_OB));`  
中的 `new(LOGIN_OB)`来建立
4. 自动调用LOGIN_OB中的logon() 方法
```
void logon()
{
    call_out("time_out", 60);

    if (interactive(this_object()))
        set_temp("ip_number", query_ip_number(this_object()));

    LOGIN_D->login(this_object());
}
```
logon()中的`LOGIN_D->login(this_object());` 调用静态对象LOGIN_D中的login方法，其中的输入参数是连线对象本身
5. login()方法调用welcome() 方法
6. welcome() 方法中`color_cat(MOTD);` 实现登录界面图像的绘制
```
    write("\n^_^!请输入你的登录ID:");
    input_to("get_id", ob);   
```
开始具体的登录交互
   
注意几点：
1. 代码中的类也就是.c文件的路径在/include/mudcore.h中定义
```
  /* 核心对象 */
#define CORE_MASTER_OB      CORE_DIR "system/kernel/master"
#define CORE_SIMUL_EFUN_OB  CORE_DIR "system/kernel/simul_efun"
#define CORE_VOID_OB        CORE_DIR "system/object/void"
#define CORE_LOGIN_OB       CORE_DIR "system/object/login"
#define CORE_USER_OB        CORE_DIR "system/object/user"
/* 守护进程 */
#define CORE_AREA_PATTERN_D CORE_DIR "system/daemons/area_pattern_d"
#define CORE_CHANNEL_D      CORE_DIR "system/daemons/channel_d"
#define CORE_CHAR_D         CORE_DIR "system/daemons/char_d"
#define CORE_CHINESE_D      CORE_DIR "system/daemons/chinese_d"
#define CORE_COMBAT_D       CORE_DIR "system/daemons/combat_d"
#define CORE_COMMAND_D      CORE_DIR "system/daemons/command_d"
#define CORE_DBASE_D        CORE_DIR "system/daemons/dbase_d"
#define CORE_EMOTE_D        CORE_DIR "system/daemons/emote_d"
#define CORE_ENV_D          CORE_DIR "system/daemons/env_d"
#define CORE_LOGIN_D        CORE_DIR "system/daemons/login_d"
#define CORE_NAME_D         CORE_DIR "system/daemons/name_d"
#define CORE_NATURE_D       CORE_DIR "system/daemons/nature_d"
#define CORE_QUEST_D        CORE_DIR "system/daemons/quest_d"
#define CORE_TIME_D         CORE_DIR "system/daemons/time_d"
#define CORE_VERB_D         CORE_DIR "system/daemons/verb_d"
#define CORE_VIRTUAL_D      CORE_DIR "system/daemons/virtual_d"
```
2. 其中涉及了两个Apply函数[connect](https://mud.wiki/Connect)和[logon](https://mud.wiki/Logon)