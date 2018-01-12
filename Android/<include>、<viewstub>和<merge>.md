#\<include\>、\<viewstub\>和\<merge\>
###\<incldue\>标签
include标签常用于将布局中的公共部分提取出来供其他layout共用，需要layout属性，如果设置了宽高会覆盖引入布局根节点的宽高，设置了id会覆盖引入布局根布局的id



###\<viewstub\>标签 
viewstub引入的布局默认不会扩张，即既不会占用显示也不会占用位置，从而在解析layout时节省cpu和内存，viewstub常用来引入那些默认不会显示，只在特殊情况下显示的布局，如进度布局、网络失败显示的刷新布局、信息出错出现的提示布局等。        
<p>在代码中，获取到ViewStub后调用inflate方法来展开布局

###\<merge\>标签			
减少布局层级，可用于两种典型情况：

1. 布局顶节点是FrameLayout且不需要设置背景或padding等属性
2. 某布局作为自布局被其他布局include时，使用merge做该布局的顶节点，被引入时顶节点被自动忽略
