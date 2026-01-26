## RNN
### 类别
#### 单向
![[Pasted image 20251206195938.png]]
![[Pasted image 20251206195917.png]]
>也就是不同之处在于，Jordan Network把输出值放进去，而前者是某个隐藏层的输出值的存储结果。
#### 双向（Bidirectional RNN）
![[Pasted image 20251206200521.png]]
##### 好处
不仅能看到 $x^t$ 之前的，也能看到之后的，所以能得到更多的依赖信息。

---

## LSTM
![[Pasted image 20251206201236.png]]![[Pasted image 20251206202801.png]]
>这里面，三个门的激活函数一般都使用`Sigmoid`。
### 结构
- Input Gate: 控制网络的输入多大程度影响输出；
- Output Gate: 决定最后输出；
- Forget Gate: 决定上一个输入（图中是`c`）多大程度上影响本次的输出；
### 运算示例
![[Pasted image 20251206204038.png]]
假设$g$和$h$都是$y=x$形式的，这样计算较为方便。
可以自行计算。
![[Pasted image 20251206205226.png]]
### 规范形态
![[Pasted image 20251206210620.png]]
>在每一层输入的时候，不仅要对输入的token作线性变换，还有考虑上一层的输出和Memory Cell的值（$c^{t-1}, h^{t-1}$），三者拼起来一起做线性变换，才是真正的四个输入到网络中的量。