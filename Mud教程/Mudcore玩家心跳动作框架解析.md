<!--
 * @Author: Donald duck tang5722917@163.com
 * @Date: 2023-03-27 19:15:06
 * @LastEditors: Donald duck tang5722917@163.com
 * @LastEditTime: 2023-03-27 19:35:23
 * @FilePath: \mysticism-mud-doc\Mud教程\Mudcore玩家心跳动作框架解析.md
 * @Description:
 * Copyright (c) 2023 by Donald duck email: tang5722917@163.com, All Rights Reserved.
-->
# MudCore玩家心跳框架解析（学习笔记）
心跳--heart_beat()是MUD游戏控制的核心apply,一切玩家/NPC随时间自行动作都要通过这个apply来实现。  
而mudcore中提供了一个框架接口来集中的定义/控制玩家/NPC的apply。  
这里我梳理一下mudcore中heart_beat()接口框架的实现与功能。  
mudcore中与heart_beat()接口框架相关的代码文件由三个:  
```
mudcore/system/object/user.c  //heart_beat() apply实现
mudcore/inherit/condition.c   //心跳事件的框架，提供相应的方法，living会继承这个类
mudcore/inherit/condition_mod.c //提供心跳事件的公共接口，游戏心跳事件继承自这个类
```
## 1 user.c中定义的heart_beat()方法  
以下为heart_beat()代码，我们来逐行理解  
```C
// 玩家心跳事件
void heart_beat()
{
    mapping condition;

    if (mapp(condition = query("condition")))
    {
        foreach (string key, mapping value in condition)
        {
            if (value["time"] <= 0)
            {
                catch(replace_string(key, "#", "/")->stop_effect(this_object()));
                delete("condition/" + key);
                continue;
            }

            if (value["heart_beat"] > 0 && !(value["time"] % value["heart_beat"]))
                catch(replace_string(key, "#", "/")->heart_beat_effect(this_object()));

            add("condition/" + key + "/time", -1);
        }
    }
}
```

## 2 
