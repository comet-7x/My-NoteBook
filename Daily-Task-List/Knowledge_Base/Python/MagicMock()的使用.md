
|源代码写法|Mock 赋值方式|解释|
|---|---|---|
|obj.attr（无括号）|mock.attr = "值"|模拟普通属性 / 变量|
|obj.method ()（有括号）|mock.method.return_value = "值"|模拟函数 / 方法的调用结果|
|func ()（顶层函数）|mock_func.return_value = "值"|模拟函数的调用结果|
|obj.a.b.c（链式）|mock.a.b.c = "值"|模拟深层属性|
|obj.method().attr|mock.method.return_value.attr = "值"|模拟方法返回值的属性|
