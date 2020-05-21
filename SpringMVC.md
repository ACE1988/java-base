## SpringMVC 流程
1.  用户发送请求至前端控制器DispatcherServlet。
2.  DispatcherServlet收到请求调用HandlerMapping处理器映射器。
3.  处理器映射器找到具体的处理器(可以根据xml配置、注解进行查找)，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。
4.  DispatcherServlet调用HandlerAdapter处理器适配器。
5.  HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。
6.  Controller执行完成返回ModelAndView。
7.  HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet。
8.  DispatcherServlet将ModelAndView传给ViewReslover视图解析器。
9.  ViewReslover解析后返回具体View。
10. DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。
11. DispatcherServlet响应用户。

## 组件说明：
以下组件通常使用框架提供实现：
. DispatcherServlet：作为前端控制器，整个流程控制的中心，控制其它组件执行，统一调度，降低组件之间的耦合性，提高每个组件的扩展性。
. HandlerMapping：通过扩展处理器映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。 
. HandlAdapter：通过扩展处理器适配器，支持更多类型的处理器。
. ViewResolver：通过扩展视图解析器，支持更多类型的视图解析，例如：jsp、freemarker、pdf、excel等。
