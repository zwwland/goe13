## Mac 下搭建 Common Lisp 开发环境 
1. 安装 emacs
`brew install --with-cocoa --srgb emacs`
2. 安装 sbcl
`brew install sbcl`
3. 安装 slime，这个需要配置 [EMLPA 源](https://mirrors.tuna.tsinghua.edu.cn/help/elpa/)，之后在 Emacs 里面安装
`M-x package-install RET slime RET`
4. 修改 emacs 配置文件
```lisp
(setq inferior-lisp-program "/usr/local/bin/sbcl")
(setq slime-contribs '(slime-fancy))
(require 'slime-autoloads)
```
5. 启动
`M-x RET slime RET`

## common lisp - 函数

函数不仅是 Lisp 程序的根基,它同时也是 Lisp 语言的基石。

除了少数称为特殊形式 (special form) 的操作符之外，Lisp 的核心就是一个函数的集合。如果你觉得某件事 Lisp 应该能做，你完全可以把它写出来，然后你的新函数可以享有和内置函数同等的待遇。

有两点让 Lisp 函数与众不同。一是Lisp 本身就是函数的集合。这意味着我们可以自己给 Lisp 增加新的操作符。另一个就是:函数也是 Lisp 的对象。这意味着在 Lisp 里我们可以像对待其他熟悉的数据类型那样来对待函数,就像整数那样:在运行期创建一个新函数,把函数保存在变量和结构体里面,把它作为参数传给其他函数,还有把它作为函数的返回值。

### 定义函数
最通常的做法就是用defun宏实现函数定义，如下面定义的double函数。
```lisp
> (defun double (x) (* 2 x))
DOUBLE
```
函数本身就是对象，defun就是构造这样的对象，然后保存它到第一个参数名下。所以我们既可以调用double，也可以持有这个名字对应的函数对象。通常可以用#'操作符得到这个对象，这个操作符能将名字映射到实际的函数对象。就像下面这样：

```lisp
> #'double
#<FUNCTION DOUBLE>
```
也可以使用On Lisp

可以使用一种称为 λ–表达式 (lambda-expression) 的东西定义函数，它由三部分组成：lambda符号，参数列表和若干表达式主体。下面的λ–表达式是等价于double的函数：

```lisp
> (lambda (x) (* 2 x))
```
λ–表达式也可以看作是函数的名字，如果说 double 是个正规的名字，那么lambda表达式就相当于具体的描述。通过#'可以得到相应的函数：

```lisp
> #'(lambda (x) (* 2 x))
#<FUNCTION (LAMBDA (X)) {100366DEAB}>
```
由于 λ–表达式同时也是函数的名字,因而它也可以出现在函数调用的最前面：

```lisp
> ((lambda (x) (* x 2)) 3)
6
```
在 Common Lisp 里,我们可以同时拥有名为 double 的函数和变量。

"拥有分开的变量和函数命名空间的 Lisp 称为 Lisp–2,在另一类 Lisp–1 下,变量和函数定义在同一命名空间里,最著名的这种 Lisp 方言是 Scheme。关于 Lisp–1 vs. Lisp–2 在网上有很多讨论,一般观点认为 Lisp–1 对于编译器来说更难实现。"

```lisp
> (setq double 2)
2
> (double double)
4
```
当符号出现在函数调用的首位，或者前置 #’ 的时候，它被认为是函数。其他场合下它被当成变量名。

Common Lisp 还提供了两个函数用于将符号映射到它所代表的函数或者变量，以备不时之需。

symbol-value 函数以一个符号为参数，返回对应变量的值：

```lisp
> (symbol-value 'double)
2
```
而symbol-function则可以得到一个全局定义的函数：

```lisp
> (symbol-function 'double)
#<FUNCTION DOUBLE>
```
由于函数也是对象，变量也可以把函数当作值

```lisp
(setf double-1 #'double)
```
其实，defun实际上是把它的第一个参数的symbol-function设置成了用它其余部分构造的函数，下面两个表达式完成的功能基本相同：

```lisp
(defun double (x) (* 2 x))

(setf (symbol-function 'double) #'(lambda (x) (* 2 x)))
```
### 函数型参数
函数同为数据对象，就意味着我们可以像对待其他对象那样把它传递给其他函数。common lisp中提供了apply和funcall让我们实现对函数的调用：

```lisp
(+ 1 2)
(apply #’+ ’(1 2))
(apply (symbol-function ’+) ’(1 2))
(apply #’(lambda (x y) (+ x y)) ’(1 2))
(apply #’+ 1 ’(2))
(funcall #’+ 1 2)
```
apply和funcall的唯一区别就是，apply必须以列表参数结尾。

### 作为属性的函数
函数作为 Lisp对象这一事实也创造了条件，让我们能够编写出那种可以随时扩展以满足新需求的程序。假设我们需要写一个以动物种类作为参数并产生相应行为的函数。在大多数语言中，会使用 case 语句达到这个目的，Lisp 里也可以用同样的办法：

```lisp
(defun behave (animal) 
  (case animal
    (dog (wag-tail) (bark))
    (rat (scurry) (squeak))
    (cat (rub-legs) (scratch-carpet))))
```
如果把每种个体动物的行为以单独的函数形式保存，会更利于扩展：

```lisp
(defun behave (animal) (funcall (get animal ’behavior)))

(setf (get ’dog ’behavior) 
  #’(lambda () (wag-tail) (bark)))
```
### 作用域
Common Lisp 是一种词法作用域 (lexically scope) 的 Lisp方言，词法作用域与动态作用域的区别在于语言处理自由变量的方式不同。

当一个符号被用来表达变量时，我们称这个符号在表达式中是被绑定（bound）的，这里的变量可以是参数，也可以来自于像let，do这样的变量绑定操作符。如果符号不受到约束，就认为它是自由的，比如：

```lisp
(let ((y 7))
  (defun scope-test (x)
    (list x y)))
```
在函数表达式里，x是受约束的，而y是自由的。自由变量有意思的地方就在于，这种变量应有的值并不那么显而易见。一个约束变量的值是确信无疑的，当调用 scope-test 时，x 的值就是通过参数传给它的值。但 y 的值应该是什么呢？这要看具体方言的作用域规则。

在 动态作用域 的 Lisp 里，要想找出当 scope-test 执行时自由变量的值，我们要往回逐个检查函数的调用链。当发现 y 被绑定时，这个被绑定的值即被用在 scope-test 中。如果没有发现，那就取 y 的全局值。这样，在用动态作用域的 Lisp 里，在调用的时候 y 将会产生这样的值：

```lisp
> (let ((y 5)) (scope-test 3))
(3 5)
```
在 词法作用域 的 Lisp 里，我们不再往回逐个检查函数的调用链，而是逐层检查定义这个函数时，它所处的各层外部环境。在一个词法作用域 Lisp 里，我们的示例将捕捉到定义 scope-test 时，变量 y 的绑定。所以可以在 Common Lisp 里观察到下面的现象：

```lisp
 > (let ((y 5)) (scope-test 3))
(3 7)
```
### 闭包
common lisp是词法作用域，如果定义含有自由变量的函数，系统就必须在函数定义时保存那些变量的绑定。这种函数和一组变量绑定的组合称为 闭包。闭包是带有局部状态的函数。下面列举一些这样的例子：

```lisp
(defun list+ (lst n)
  (mapcar #’(lambda (x) (+ x n))
        lst))

> (list+ ’(1 2 3) 10) 
(11 12 13)
```

```lisp
(defun make-adderb (n)
  #’(lambda (x &optional change)
      (if change
          (setq n x)
      (+ x n))))

> (setq addx (make-adderb 1))

> (funcall addx 3)
4

> (funcall addx 100 t)
100

> (funcall addx 3)
103
```
### 局部函数
在用lambda表达式时，由于它没有自己的名字，因此没办法引用自己，这就意味着在common lisp里，不能用lambda表达式定义递归函数。

如果我们想把某个列表上的参数一一应用到某个函数上，可以这样写：

```lisp
> (mapcar #'(lambda (x) (+ 2 x))
    '(1 2 3 4))

(3 4 5 6)
```
要是想把递归函数作为第一个参数送给 mapcar 呢？如果函数已经用 defun 定义了，我们就可以通过名
字简单地引用它：

```lisp
 > (mapcar #’copy-tree ’((a b) (c d e))) 

((A B) (C D E))
```
但现在假设这个函数必须是一个闭包，它从 mapcar 所处的环境获得绑定，比如：

```lisp
(defun list+ (lst n)
  (mapcar #’(lambda (x) (+ x n))
        lst))
```
mapcar 的第一个参数是 #’(lambda (x) (+ x n))，它必须要在 list+ 里定义，原因是它需要捕捉 n 的绑定。到目前为止都还一切正常，但如果要给 mapcar 传递一个函数，而这个函数在需要局部绑定的同时也是递归的呢？我们不能使用一个在其他地方通过 defun 定义的函数，因为这需要局部环境的绑定。并且我们也不能使用 lambda 来定义一个递归函数，因为这个函数将无法引用其自身。

Common Lisp 提供了 labels 帮助我们跳出这个两难的困境。除了在一个重要方面有所保留外，labels 基本可以看作是 let 的函数版本。labels 表达式里的每个绑定规范都必须符合如下形式：

```lisp
(⟨name⟩ ⟨parameters⟩ . ⟨body⟩)
```
在 labels 表达式里，⟨name⟩ 将指向与下面表达式等价的函数：

```lisp
#’(lambda ⟨parameters⟩ . ⟨body⟩)
```
例如：

```lisp
> (labels ((inc (x) (1+ x))) 
    (inc 3))
> 4
```
尽管如此，在 let 与 labels 之间有一个重要的区别。在 let 表达式里,变量的值不能依赖于同一
个 let 里生成的另一个变量，就是说，你不能这样写( let*可以这样写 )：

```lisp
(let ((x 10) 
       (y x))
  y)
```
然后认为这个新的 y 能反映出那个新 x 的值。相反，在 labels 里定义的函数 f 的函数体里就可以引用那里定义的其他函数，包括 f 本身，这就使定义递归函数成为可能。

使用 labels，我们就可以写出类似 list+ 这样的函数了，但这里 mapcar 的第一个参数是递归函数：

```lisp
(defun count-instances (obj lsts)
  (labels ((instances-in (lst)
             (if (consp lst)
                 (+ (if (eq (car lst) obj) 1 0)
                    (instances-in (cdr lst)))
                 0)))
    (mapcar #’instances-in lsts)))
```
该函数接受一个对象和一个列表，然后分别统计出该对象在列表的每个元素 (作为列表) 中出现的次数，把这些次数组成列表，并返回它：

```lisp
> (count-instances ’a ’((a b c) (d a r p a) (d a r) (a a)))
(1 2 1 2)
```
### 尾递归
递归函数自己调用自己。如果函数调用自己之后不做其他工作，这种调用就称为 尾递归 (tail-
recursive)。下面这个函数不是尾递归的：

```lisp
(defun our-length (lst)
  (if (null lst)
      0
      (1+ (out-length (cdr lst)))))
```
因为在从递归调用返回之后，我们又把结果传给了 1+。

尾递归是一种令人青睐的特性，因为许多 Common Lisp 编译器都可以把尾递归转化成循环。若使用这种编译器，你就可以在源代码里书写优雅的递归，而不必担心函数调用在运行期产生的系统开销。如果一个函数不是尾递归的话，常常可以把一个使用累积器 (accumulator) 的局部函数嵌入到其中，用这种方法把它转换成尾递归的形式。在这里，累积器指的是一个参数，它代表着到目前为止计算得到的值。例如 our-length 可以转换成

```lisp
(defun our-length (lst)
  (labels ((rec (lst acc)
         (if (null lst)
         acc
         (rec (cdr lst) (1+ acc)))))
    (rec lst 0)))
```

## 在ubuntu中，升级emacs到最新版本（非源码安装）
当使用低版本的ubuntu时，安装的emacs也是低版本的，如果想使用高版本的emacs，怎么办呢？
当然可以下载emacs的源码，编译安装，有没有更省事的办法吗？

可以使用下面的方法：
```bash
$sudo add-apt-repository ppa:ubuntu-elisp/ppa
$sudo apt update
$sudo apt install emacs-snapshot emacs-snapshot-el

$emacs-snapshot --version    #查看emacs版本
```
成功安装后，原来版本的emacs还是可以通过$ emacs使用;使用新安装的emacs：$ emacs-snapshot。

