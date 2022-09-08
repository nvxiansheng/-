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
