<!--
 * @Author: Donald duck tang5722917@163.com
 * @Date: 2023-02-22 11:17:13
 * @LastEditors: Donald duck tang5722917@163.com
 * @LastEditTime: 2023-02-22 14:50:50
 * @FilePath: \mysticism-mud-doc\Mud教程\Mudcore玩家登录初始化流程.md
 * @Description: 
 * 
 * 
 * Copyright (c) 2023 by Donald duck tang5722917@163.com, All Rights Reserved. 
-->

# MudCore玩家登录初始化流程（学习笔记）  
上接 <https://bbs.mud.ren/threads/312> 
  
1. `protected void get_id(string arg, object ob)` 函数开始对玩家的账号/密码进行验证  

第一个参数传入用户输入的账号，第二个参数传入当前连接的对象  
此时需注意，目前的所有操作都是在登陆时master.c中调用connect方法返回的object中  
而这的对象来自login.c 类的实例化。  
在login.c中有代码  
   ```
   inherit _DBASE;
   inherit _SAVE;
   ```
根据mudcore.h 中：
```
#define CORE_DBASE          CORE_DIR "inherit/dbase"
#define CORE_SAVE           CORE_DIR "inherit/save"

#ifndef _DBASE
#define _DBASE          CORE_DBSAVE
#endif

#ifndef _SAVE
#define _SAVE           CORE_SAVE
#endif
```
这说明LOGIN类多继承了DBASE和SAVE类，使LOGIN类的对象可以使用DBASE和SAVE中的方法实现存/读档  
之后这两个类的方法将高频出现。DBASE类中的核心是：  
```
// 存档数据
mapping dbase;
```
DBASE提供各种方法，通过使用这些方法将角色的各种数值存储在 mapping dbase 中。  

2. get_id 函数中首先对输入的字符串大写转换为小写并进行检查，符合要求继续处理，不符合则提示重新输入
3. get_id 函数中调用如下代码：
```
    if ((string)ob->set("id", arg) != arg)
    {
        write("Failed setting user name.\n");
        destruct(ob);
        return;
    }
```
这段代码使用了DBASE中的`mixed set(string prop, mixed data)`方法  
set 方法功能为将data 存储在`mapping dbase` 中的prop项中，如果成功则返回data  
`if ((string)ob->set("id", arg) != arg)`将输入的字符串存储在dbase 的id项中  

4. get_id()函数中： `if (file_size(ob->query_save_file() + __SAVE_EXTENSION__) >= 0)`  

这段代码来验证输入的账号是否注册过  
其中`query_save_file()`是LOGIN类的一个成员函数：  
```
string query_save_file()
{
    string id;

    id = query("id", 1);
    if (! stringp(id)) return 0;
    return sprintf(DATA_DIR "login/%s", id);
}
```
这段代码中`id = query("id", 1)`的功能是返回dbase中'ID'项第一个数据，并赋值给id（其实就是刚才 3 中刚存储进去的）  
`return sprintf(DATA_DIR "login/%s", id);` 将输入的id组合成一个文件路径作为query_save_file()的返回值  
mudcore默认为\data\login，里面的*.o文件就是每个账号的存档  
`__SAVE_EXTENSION__`编译时开启`DEBUG`后有效,现在可以先无视  
[filesize()](https://mud.wiki/File_size)是一个Apply 方法，用来返回输入文件路径的文件大小。  
5. 通过 `ob->restore()` 来读取对应id的存储  
6. 调用`get_passwd(string pass, object ob)`来进行密码验证  
其中第一个参数是输入的密码字符串，第二个参数依旧传入当前连接的对象  
7. 密码验证成功之后调用 `check_ok(ob);`开始对角色进行初始化操作  
这个函数的输入参数'ob'旧是当前连接的对象 

