4. 函数
------------

4.1. 输入和输出
~~~~~~~~~~~~~~~~~~~~~~~~

**总述**

我们倾向于按值返回， 否则按引用返回。 避免返回指针， 除非它可以为空. 

**说明**

C++ 函数由返回值提供天然的输出， 有时也通过输出参数（或输入/输出参数）提供. 我们倾向于使用返回值而不是输出参数： 它们提高了可读性， 并且通常提供相同或更好的性能. 

C/C++ 中的函数参数或者是函数的输入, 或者是函数的输出, 或兼而有之. 非可选输入参数通常是值参或 ``const`` 引用, 非可选输出参数或输入/输出参数通常应该是引用 （不能为空）. 对于可选的参数， 通常使用 ``std::optional`` 来表示可选的按值输入， 使用 ``const`` 指针来表示可选的其他输入． 使用非常量指针来表示可选输出和可选输入/输出参数．

避免定义需要 ``const`` 引用参数去超出生命周期的函数， 因为 ``const`` 引用参数与临时变量绑定． 相反， 要找到某种方法来消除生命周期要求 （例如， 通过复制参数）， 或者通过 ``const`` 指针传递它并记录生命周期和非空要求.

在排序函数参数时， 将所有输入参数放在所有输出参数之前. 特别要注意, 在加入新参数时不要因为它们是新参数就置于参数列表最后, 而是仍然要按照前述的规则, 即将新的输入参数也置于输出参数之前.

这并非一个硬性规定. 输入/输出参数 (通常是类或结构体) 让这个问题变得复杂. 并且, 有时候为了其他函数保持一致, 你可能不得不有所变通.

4.2. 编写简短函数
~~~~~~~~~~~~~~~~~~~~~~~~

**总述**

我们倾向于编写简短, 凝练的函数.

**说明**

我们承认长函数有时是合理的, 因此并不硬性限制函数的长度. 如果函数超过 40 行, 可以思索一下能不能在不影响程序结构的前提下对其进行分割.

即使一个长函数现在工作的非常好, 一旦有人对其修改, 有可能出现新的问题, 甚至导致难以发现的 bug. 使函数尽量简短, 以便于他人阅读和修改代码.

在处理代码时, 你可能会发现复杂的长函数. 不要害怕修改现有代码: 如果证实这些代码使用 / 调试起来很困难, 或者你只需要使用其中的一小段代码, 考虑将其分割为更加简短并易于管理的若干函数.

4.3. 函数重载
~~~~~~~~~~~~~~~~~~~~~~

**总述**

若要使用函数重载, 则必须能让读者一看调用点就胸有成竹, 而不用花心思猜测调用的重载函数到底是哪一种. 这一规则也适用于构造函数.

**定义**

你可以编写一个参数类型为 ``const string&`` 的函数, 然后用另一个参数类型为 ``const char*`` 的函数对其进行重载:

.. code-block:: c++

    class MyClass {
        public:
        void Analyze(const string &text);
        void Analyze(const char *text, size_t textlen);
    };

**优点**

通过重载参数不同的同名函数, 可以令代码更加直观. 模板化代码需要重载, 这同时也能为使用者带来便利.

**缺点**

如果函数单靠不同的参数类型而重载 (acgtyrant 注：这意味着参数数量不变), 读者就得十分熟悉 C++ 五花八门的匹配规则, 以了解匹配过程具体到底如何. 另外, 如果派生类只重载了某个函数的部分变体, 继承语义就容易令人困惑.

**结论**

如果打算重载一个函数, 可以试试改在函数名里加上参数信息. 例如, 用 ``AppendString()`` 和 ``AppendInt()`` 等, 而不是一口气重载多个 ``Append()``. 如果重载函数的目的是为了支持不同数量的同一类型参数, 则优先考虑使用 ``std::vector`` 以便使用者可以用 :ref:`列表初始化 <braced-initializer-list>` 指定参数.

4.4. 缺省参数
~~~~~~~~~~~~~~~~~~~~~~

**总述**

只允许在非虚函数中使用缺省参数, 且必须保证缺省参数的值始终一致. 缺省参数与 :ref:`函数重载 <function-overloading>` 遵循同样的规则. 一般情况下建议使用函数重载, 尤其是在缺省函数带来的可读性提升不能弥补下文中所提到的缺点的情况下.

**优点**

有些函数一般情况下使用默认参数, 但有时需要又使用非默认的参数. 缺省参数为这样的情形提供了便利, 使程序员不需要为了极少的例外情况编写大量的函数. 和函数重载相比, 缺省参数的语法更简洁明了, 减少了大量的样板代码, 也更好地区别了 "必要参数" 和 "可选参数".

**缺点**

缺省参数实际上是函数重载语义的另一种实现方式, 因此所有 :ref:`不应当使用函数重载的理由 <function-overloading>` 也都适用于缺省参数.

虚函数调用的缺省参数取决于目标对象的静态类型, 此时无法保证给定函数的所有重载声明的都是同样的缺省参数.

缺省参数是在每个调用点都要进行重新求值的, 这会造成生成的代码迅速膨胀. 作为读者, 一般来说也更希望缺省的参数在声明时就已经被固定了, 而不是在每次调用时都可能会有不同的取值.

缺省参数会干扰函数指针, 导致函数签名与调用点的签名不一致. 而函数重载不会导致这样的问题.

**结论**

对于虚函数, 不允许使用缺省参数, 因为在虚函数中缺省参数不一定能正常工作. 如果在每个调用点缺省参数的值都有可能不同, 在这种情况下缺省函数也不允许使用. (例如, 不要写像 ``void f(int n = counter++);`` 这样的代码.)

在其他情况下, 如果缺省参数对可读性的提升远远超过了以上提及的缺点的话, 可以使用缺省参数. 如果仍有疑惑, 就使用函数重载.

4.5. 函数返回类型后置语法
~~~~~~~~~~~~~~~~~~~~~~~~~

**总述**

只有在常规写法 (返回类型前置) 不便于书写或不便于阅读时使用返回类型后置语法.

**定义**

C++ 现在允许两种不同的函数声明方式. 以往的写法是将返回类型置于函数名之前. 例如:

.. code-block:: c++

    int foo(int x);

C++11 引入了这一新的形式. 现在可以在函数名前使用 ``auto`` 关键字, 在参数列表之后后置返回类型. 例如:

.. code-block:: c++

    auto foo(int x) -> int;

后置返回类型为函数作用域. 对于像 ``int`` 这样简单的类型, 两种写法没有区别. 但对于复杂的情况, 例如类域中的类型声明或者以函数参数的形式书写的类型, 写法的不同会造成区别.

**优点**

后置返回类型是显式地指定 :ref:`Lambda 表达式 <lambda-expressions>` 的返回值的唯一方式. 某些情况下, 编译器可以自动推导出 Lambda 表达式的返回类型, 但并不是在所有的情况下都能实现. 即使编译器能够自动推导, 显式地指定返回类型也能让读者更明了.

有时在已经出现了的函数参数列表之后指定返回类型, 能够让书写更简单, 也更易读, 尤其是在返回类型依赖于模板参数时. 例如:

.. code-block:: c++

    template <class T, class U> auto add(T t, U u) -> decltype(t + u);

对比下面的例子:

.. code-block:: c++

    template <class T, class U> decltype(declval<T&>() + declval<U&>()) add(T t, U u);

**缺点**

后置返回类型相对来说是非常新的语法, 而且在 C 和 Java 中都没有相似的写法, 因此可能对读者来说比较陌生.

在已有的代码中有大量的函数声明, 你不可能把它们都用新的语法重写一遍. 因此实际的做法只能是使用旧的语法或者新旧混用. 在这种情况下, 只使用一种版本是相对来说更规整的形式.

**结论**

在大部分情况下, 应当继续使用以往的函数声明写法, 即将返回类型置于函数名前. 只有在必需的时候 (如 Lambda 表达式) 或者使用后置语法能够简化书写并且提高易读性的时候才使用新的返回类型后置语法. 但是后一种情况一般来说是很少见的, 大部分时候都出现在相当复杂的模板代码中, 而多数情况下不鼓励写这样 :ref:`复杂的模板代码 <template-metaprogramming>`.




.. contents:: Table of Contents
   :depth: 3
   :local:
   