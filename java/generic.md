- > 泛型参数不构成父子关系，List<Integer> is not subtype of List<Number>
- > 带泛参的方法，调用方可以显式声明参数类型也可以省略，有编译器推断。但在jdk<=7下，有的地方需要用到，根据方法参数类型推断类型：void test(List<String> ss)，
如果test(Collections.emptyList())则编译不通过，需test(Collection.<String>emptyList())。在jdk>=8下，会根据方法参数推断，即：test(Collections.emptyList())
