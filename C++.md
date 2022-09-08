# 右值引用、移动语义和完美转发
### move失效
move会保留const特性
如果T是个左值引用，那么T&&由于是个万能引用，因此T&&就成了左值引用。remove_reference就可以将引用型别取消。
并不一定保证移动会产生而非复制。
```
class Annotation{
public:
   explicit Annotation(const string& text):text_(std::move(text)) {}
private:
    string text_;
}
```
text本身是个const引用，就算move后也是个const右值，考察string构造函数。指涉到的常量的左值引用允许帮当到一个常量右值型别的形参，会调用复制构造函数。
1.想要取得对某个对象执行移动操作的能力，不要将其声明为常量。  
万能引用：有型别推导，为万能引用。加了一个const或者部分已知的模板类型都是右值引用。
```
template<typename T>
void f(T&& param);  // param 是万能引用

template<typename MyTemplateType>
void f(MyTemplateType&& param);  // param 是万能引用

auto&& var2 = var1;  // var2 是万能引用
```
```
template<typename T>
void f(std::vector<T>&& param);  // param 是右值引用

std::vector<int> v;
f(v);  // 报错，因为要求传入右值

template<typename T>
void f(const T&& param);  // param 是右值引用
```
push_back中的T实际上不需要型别推导，完全跟着类的T模板编译走的。
```
template<class T,class Allocator=allocator<T>>
class vector{
public:
   void puah_back(T&& x);
   template<class... Args>
   void emplace_back(Args&&... args);//万能引用
};
```
C++14更好万能引用auto&&
```
auto timeFuncInvocation=
   [](auto&& func,auto&&... params)
   {
      std::forward<decltype(func)>(func)(std::forward<decltype(params)>(params)...);
      //计时器停止，记录花费时间
   }
   ```
   
 ### 针对右值引用实施std::move,针对万能引用实施std::forward
 ```
 class Widget {
public:
    Widget(Widget&& rhs)
    : name(std::move(rhs.name)),
      p(std::move(rhs.p))
    {}
private:
    std::string name;
    std::shared_ptr<SomeDataStructure> p;
};
```
```
// 按值返回
Matrix
operator+(Matrix&& lhs, const Matrix& rhs)
{
    lhs += rhs;
    return std::move(lhs);  // 将 lhs 移入返回值
}  // 如果直接 return lhs ，则会将左值 lhs 拷贝到返回值存储位置
```
## 返回值优化
直接在为函数返回值分配的内存上创建局部变量w来避免复制。
1.局部对象型别和函数返回值型别相同  
2.返回的就是局部对象本身
