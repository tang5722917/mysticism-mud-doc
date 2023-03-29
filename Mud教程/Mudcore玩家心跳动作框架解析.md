<!--
 * @Author: Donald duck tang5722917@163.com
 * @Date: 2023-03-27 19:15:06
 * @LastEditors: Donald duck tang5722917@163.com
 * @LastEditTime: 2023-03-29 19:24:09
 * @FilePath: \mysticism-mud-doc\Mud教程\Mudcore玩家心跳动作框架解析.md
 * @Description:
 * Copyright (c) 2023 by Donald duck email: tang5722917@163.com, All Rights Reserved.
-->
# MudCore心跳框架解析（学习笔记）
心跳--heart_beat()是MUD游戏控制的核心apply,一切玩家/NPC随时间自行动作都要通过这个apply来实现。  
而mudcore中提供了一个框架接口来集中的定义/控制玩家/NPC的apply。  
这里我梳理一下mudcore中heart_beat()接口框架的实现与功能。  
mudcore中与heart_beat()接口框架相关的代码文件由三个:  
```
mudcore/system/object/user.c  //heart_beat() apply实现
mudcore/inherit/condition.c   //心跳事件的框架，提供相应的方法，living会继承这个类
mudcore/inherit/condition_mod.c //提供心跳事件的公共接口，游戏心跳事件继承自这个类
```
通过一个例子来看如何使用mudcore的心跳框架  
功能是登录后实现一个倒计时  
## 实现一个USER启动倒计时  
首先建立一个心跳事件类test.c,代码如下：
```C
#include <ansi.h>
inherit _CONDITION_MOD; //心跳事件必须继承condition_mod.c
int num; 

string name="计时器测试";//定义心跳事件名称
string type="测试";     //定义心跳事件类型
string id="000";       //定义心跳事件ID
                       //以上三个属性必须定义，否则会报错
// 进入状态时的效果
void start_effect(object ob)  //覆盖condition_mod中的start_effect方法
{
    num=5;            //倒计时初始为5
    msg("MAG", "$ME进入了「" + query_condition_name() + MAG "」的" + query_condition_type() + "状态。", ob);
}
//每次心跳执行一次
void heart_beat_effect(object ob)//覆盖condition_mod中的heart_beat_effect方法
{
    write("倒计时时间："+num+"\n"); //显示倒计时时间
    num-=1;                        //每次心跳倒计时减一
}
```
之后我们要把这个心跳时间加载到USER类中。  
相关的控制方法集成在condition.c类中。  
在mudcore框架中默认living类已经继承了condition，所以condition的方法在user中可以直接使用。   
下面代码是mudcore/system/login_d.c中玩家进入后初始化函数。  
代码中只有最后一行是新加入的，用来启动倒计时心跳事件。
```C
// 进入游戏
void enter_world(object ob, object user)
{
#ifdef START_ROOM
    string start_room = START_ROOM;
#else
    string start_room = VOID_OB;
#endif
    user->set_temp("login_ob", ob);
    ob->set_temp("user_ob", user);
    if (interactive(ob))
        exec(user, ob);

    user->setup(); // 激活玩家角色
    user->set("last_login_ip", ob->query_temp("ip_number"));
    user->set("last_login_at", time());
    user->set("last_saved_at", time());
    user->add("login_times", 1);
    user->save(); // 保存玩家数据
    user->move(start_room);
    tell_room(start_room, user->short() + "连线进入这个世界。\n", ({user}));
    user->start_condition(file_name(load_object(PATH "test")),5,1); //PATH之前test.c文件路径字符串
}
```
我们来看`user->start_condition(file_name(load_object(PATH "test")),5,1);`这一句。  
函数的原型是`varargs nomask void start_condition(string condition_file, int time, int heart_beat)`  
这个函数的第二/三参数很好理解，time表示一共执行次数，heart_beat表示多少个系统心跳执行一次。  
代码中的5/1就表示每秒（默认心跳为1s）执行一次，一共执行5次。  
但第一个参数非常的坑，`string condition_file`看起来是个string,但在函数里面到处都是`condition_file->`  
其中原因是LPC中对象引用的本质其实就是个字符串，具体可以看<https://bbs.mud.ren/threads/316>  
所以condition_file这个参数的定义非常坑，如果直接写`PATH "test"`它会报错，因为函数内部需要一个对象的引用，而对象必须先建立才能有引用。  
没有经过load_object或new载入.c文件的对象当然不能直接使用。  
但如果你直接写`load_object(PATH "test")`它同样会报错，因为`start_condition`第一个参数定义是string，而load_object返回的是object,  
虽然string能隐式转换为object，但编译时driver的类型检查无法通过。  
所以只能在外面套一个file_name() ╮(╯-╰)╭  
执行结果：  
![运行结果](https://github.com/tang5722917/mysticism-mud-doc/blob/github/pic/1/mudcore_condition.png "title")  
这里只涉及事件启动`start_condition`，在`condition.c`还定义心跳事件的其他控制函数，比如移除状态`stop_condition`,改变状态时间`change_condition_time`等等。  
