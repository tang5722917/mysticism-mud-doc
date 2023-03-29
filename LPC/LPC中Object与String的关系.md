今天研究mudcore代码发现了奇怪的写法  
在mudcore/inherit/condition.c有如下函数  
```
varargs nomask void start_condition(string condition_file, int time, int heart_beat)
{
    mapping condition_setup = allocate_mapping(0);
 
    // 如果已经在相同的状态内先停止先前的状态
    if (in_condition(condition_file))
        condition_file->stop_effect(this_object());
        .........
```
这个函数的参数是`string condition_file` 定义了一个字符串类型的condition_file  
但在函数内部又出现了`condition_file->stop_effect(this_object());`  
明显condition_file在这里是一个对象。   
难道LPC中 string 真的可以隐式转换为object？  
针对这个问题我做了如下实验   
```
void look(object me,object env)

{
    object o;  //定义一个object o
    string s;  //定义一个string s
    o = this_player(); //o 作为当前user的引用
    s = file_name(o);  //s 为o的文件路径
    debug_message(o->short());
    debug_message(s);
    debug_message(s->short());
}
```
这段代码的结果为
```
管理员(My)
/system/obj/user#2
管理员(My)
```
由上述结果可知o->short()等于s->short()，o=s  
也就是说一个对象的引用，也就是object类型本质就是一个字符串。  
这个字符串内容就是对象的文件名。  
LPC在这里的实现还真是简单粗暴直接^_^  
