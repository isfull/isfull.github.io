## gtest使用
1. 拉取gtest代码到一个地方https://github.com/google/googletest.git
2. gtest官方已经准备了sample和标准版的Makefile，你只要把googletest/googletest/make/目录里的Makefile改改就能用
>参考我的https://github.com/isfull/mohosojo/tree/master/weighted_random_selection
3. gtest准备了一个默认的main文件在googletest/googletest/src目录里，通过上面说的Makefile可以自动关联上，无需自己写main方法调用
4. 具体的测试代码编写方式就很简单了，大部分用TEST就够了，有些需要驱动、桩的，需要使用TEST_F
5. code.h, code.cc, code_unittest.cc Makefile一起生成code_unittest执行文件，执行就行
