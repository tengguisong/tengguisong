### 常见的error
- 编译错误解决，dereferencing pointer to incomplete type "结构体名"
-- 1.使用库函数时，没有包含相对应的头文件
-- 2.自定义结构体，将其定义在.c文件中。
除非明确说明，本文内容仅针对x86/x86_64的Linux开发环境，有朋友说baidu不到，开个贴记录一下（加粗字体是关键词）：

用“-Wl,-Bstatic”指定链接静态库，使用“-Wl,-Bdynamic”指定链接共享库，使用示例：
-Wl,-Bstatic -lmysqlclient_r -lssl -lcrypto -Wl,-Bdynamic -lrt -Wl,-Bdynamic -pthread -Wl,-Bstatic -lgtest
（"-Wl"表示是传递给链接器ld的参数，而不是编译器gcc/g++的参数。）


1) 下面是因为没有指定链接参数-lz（/usr/lib/libz.so，/usr/lib/libz.a ）
/usr/local/mysql/lib/mysql/libmysqlclient.a(my_compress.c.o): In function `my_uncompress':
/home/software/mysql-5.5.24/mysys/my_compress.c:122: undefined reference to `uncompress'
/usr/local/mysql/lib/mysql/libmysqlclient.a(my_compress.c.o): In function `my_compress_alloc':
/home/software/mysql-5.5.24/mysys/my_compress.c:71: undefined reference to `compress'




2) 下面是因为没有指定编译链接参数-pthread（注意不仅仅是-lpthraed）
/usr/local/mysql/lib/mysql/libmysqlclient.a(charset.c.o): In function `get_charset_name':
/home/zhangsan/mysql-5.5.24/mysys/charset.c:533: undefined reference to `pthread_once'




3) 下面这个是因为没有指定链接参数-lrt
/usr/local/thirdparty/curl/lib/libcurl.a(libcurl_la-timeval.o): In function `curlx_tvnow':
timeval.c:(.text+0xe9): undefined reference to `clock_gettime'




4) 下面这个是因为没有指定链接参数-ldl
/usr/local/thirdparty/openssl/lib/libcrypto.a(dso_dlfcn.o): In function `dlfcn_globallookup':
dso_dlfcn.c:(.text+0x4c): undefined reference to `dlopen'
dso_dlfcn.c:(.text+0x62): undefined reference to `dlsym'
dso_dlfcn.c:(.text+0x6c): undefined reference to `dlclose'




5) 下面这个是因为指定了链接参数-static，它的存在，要求链接的必须是静态库，而不能是共享库
ld: attempted static link of dynamic object
如果是以-L加-l方式指定，则目录下必须有.a文件存在，否则会报-l的库文件找不到：ld: cannot find -lACE




6) GCC编译遇到如下的错误，可能是因为在编译时没有指定-fPIC，记住：-fPIC即是编译参数，也是链接参数
relocation R_x86_64_32S against `vtable for CMyClass` can not be used when making a shared object




7) 下面的错误表示gcc编译时需要定义宏__STDC_FORMAT_MACROS，并且必须包含头文件inttypes.h
test.cpp:35: error: expected `)' before 'PRIu64'




8) 下面是因为在x86机器（32位）上编译没有指定编译参数-march=pentium4
../../src/common/libmooon.a(logger.o): In function `atomic_dec_and_test':
../../include/mooon/sys/atomic_gcc.h:103: undefined reference to `__sync_sub_and_fetch_4'




9) 下列错误可能是因为多了个“}”
error: expected declaration before '}' token




10) 下列错误可能是因为少了个“}”
error: expected `}' at end of input




11) 下面这个错误是编译一个共享库时，该共享库依赖的一静态库编译时没有加“-fPIC”参数，解决方法为带“-fPIC”重新编译被依赖的静态库
relocation R_X86_64_32 against `a local symbol' can not be used when making a shared object; recompile with -fPIC




12) 下面这个错误，是因为头文件中使用“_syscall0(pid_t, gettid)”不当引起的
./test.o: In function `gettid()':
./test.h:17: multiple definition of `gettid()'




正确的用法是使用"inline"或"static inline"修饰一下：
inline _syscall0(pid_t, gettid)
或
static inline _syscall0(pid_t, gettid)




当然也可以这样：
在.h头文件中：extern "C" pid_t gettid(void);
在.cpp文件中：_syscall0(pid_t, gettid)




_syscall0是一个宏，定义一个函数的实现。




13) 下列编译告警是因为一个static类型的函数未被使用
my.cpp:364: warning: 'int my_function(const cgicc::Cgicc&, const std::string&)' defined but not used




只需使用“__attribute__((unused))”修饰函数的声明即可：
static int __attribute__((unused)) my_function(const cgicc::Cgicc&, const std::string&);




14) 执行thrift的configure时遇到如下的错误（thrift-0.9.0和thrift-0.9.1遇到过）：
checking for setsockopt in -lsocket... no
checking for BN_init in -lcrypto... no
configure: error: "Error: libcrypto required."




原因可能是因为编译安装openssl时指定了--prefix，比如--prefix=/usr/local/thirdparty/openssl，可这样解决：
不指定thrift的configure的--with-openssl=/usr/local/thirdparty/openssl，改为：
CPPFLAGS="-I/usr/local/thirdparty/openssl/include" LDFLAGS="-ldl -L/usr/local/thirdparty/openssl/lib"
替代它就OK了。




15) 下面这个编译错误（表现为g++进入死循环），可能是由于缺少右大括号“}”导致的，比如定义名字空间时少了“}”：
/usr/include/c++/4.1.2/tr1/type_traits:408: error: 'size_t' is not a member of 'db_proxy::std'
/usr/include/c++/4.1.2/tr1/type_traits:408: error: 'size_t' is not a member of 'db_proxy::std'
/usr/include/c++/4.1.2/tr1/mu_iterate.h:49: error: 'std::tr1' has not been declared
/usr/include/c++/4.1.2/tr1/mu_iterate.h:49: error: 'std::tr1' has not been declared
/usr/include/c++/4.1.2/tr1/bind_iterate.h:78: error: 'std::tr1' has not been declared
/usr/include/c++/4.1.2/tr1/bind_iterate.h:78: error: 'std::tr1' has not been declared
/usr/include/c++/4.1.2/tr1/bind_iterate.h:78: error: 'std::tr1' has not been declared


16) protoc编译错误，下面错误是因为没有在.proto文件所在目录下执行：
/tmp/test.proto: File does not reside within any path specified using --proto_path (or -I).  You must specify a --proto_path which encompasses this file.  Note that the proto_path must be an exact prefix of the .proto file names -- protoc is too dumb to figure out when two paths (e.g. absolute and relative) are equivalent (it's harder than you think).
解决办法有两个：一是在.proto文件所在目录下执行protoc，二是为protoc指定参数--proto_path，参数值为.proto文件所在目录。


17) 下面这个编译错误，可能是因为在全局域内调用一个类对象的成员函数，全局域内是不能直接执行函的：
error: expected constructor, destructor, or type conversion before '->' token


18) 下面这个错误是因为没有链接OpenSSL的libcrypto库，或者使用了静态库，而顺序不对：
undefined symbol: EVP_enc_null


19) 下列是链接错误，不是编译错误，加上“-pthread”即可，注意不是“-lpthread”：
/usr/local/mysql/lib/mysql/libmysqlclient.a(charset.c.o): In function `get_charset_name':
/home/software/mysql-5.5.24/mysys/charset.c:533: undefined reference to `pthread_once'
/usr/local/mysql/lib/mysql/libmysqlclient.a(charset.c.o): In function `get_charset_number':


20) 依赖OpenSSL（假设安装在/usr/local/thirdparty/openssl-1.0.2a）时，如果
./configure --prefix=/usr/local/thirdparty/libssh2-1.6.0 --with-libssl-prefix=/usr/local/thirdparty/openssl-1.0.2a
时遇到如下错误：
configure: error: No crypto library found!
Try --with-libssl-prefix=PATH
 or --with-libgcrypt-prefix=PATH
 or --with-wincng on Windows


可以如下方法解决：
./configure --prefix=/usr/local/thirdparty/libssh2-1.6.0 --with-openssl CPPFLAGS="-I/usr/local/thirdparty/openssl-1.0.2a/include" LDFLAGS="-L/usr/local/thirdparty/openssl-1.0.2a/lib"


21) 错误“error: expected primary-expression before ‘;’ token”可能是因为如下原因：
UserInfoInternal* user_info_internal = new UserInfoInternal;
delete user_info; // 这里应当是user_info_internal


22) 下面这个编译错误的原因是定义字符串宏时没有加双引引号：
example.cpp:563:16: error: too many decimal points in number
如定义成了：#define IP1 127.0.0.1，改成#define IP1 "127.0.0.1"后问题即解决。


23) 下面这个编译错误是因为g++命令参数没写对，多了个“-”
g++: -E or -x required when input is from standard input
如：
CPPFLAGS=-pthread -
g++ -g -o $@ $^ $(CPPFLAGS) $(LDFLAGS)


24) libjson_7.6.1编译错误：
makefile:180: Extraneous text after `else' directive
makefile:183: *** only one `else' per conditional.  Stop.
解决办法：
找到makefile文件的第180行，将“else ifeq ($(BUILD_TYPE), debug)”，改成两行内嵌方式：
# BUILD_TYPE specific settings
ifeq ($(BUILD_TYPE), small)
    CXXFLAGS     = $(cxxflags_small)
else
    ifeq ($(BUILD_TYPE), debug) # 不能和else同一行，否则Makefile语法错误，不支持else ifeq
        CXXFLAGS     = $(cxxflags_debug)
        libname     := $(libname_debug)
    else
        CXXFLAGS    ?= $(cxxflags_default)
    endif
endif
可以通过设置环境变量来设置BUILD_TYPE，如：export BUILD_TYPE=debug
也可以通过环境变量来设置make install时的安装目录，如：export prefix=/usr/local/libjson


相关小知识：
在Makefile文件中，prefix=/usr和prefix?=/usr，是有区别的，前者赋值不能通过环境变量覆盖，后者则可以使用环境变量的值覆盖。


另外，请将第271行删除：
cp -rv $(srcdir)/Dependencies/ $(include_path)/$(libname_hdr)/$(srcdir)
改成：
cp -rv _internal/Dependencies/ $(include_path)/$(libname_hdr)/$(srcdir)
还有第258行前插入如一条命令：
mkdir -p $(inst_path)
否则“cp -f ./$(lib_target) $(inst_path)”，lib将成库文件名。


25） 编译gcc时，如果遇到下面这个错误，这是因为运行时找不到mpc、mpfr和gmp的so文件：
checking for x86_64-unknown-linux-gnu-nm... /data/gcc-4.8.2_src/host-x86_64-unknown-linux-gnu/gcc/nm
checking for x86_64-unknown-linux-gnu-ranlib... ranlib
checking for x86_64-unknown-linux-gnu-strip... strip
checking whether ln -s works... yes
checking for x86_64-unknown-linux-gnu-gcc... /data/gcc-4.8.2_src/host-x86_64-unknown-linux-gnu/gcc/xgcc -B/data/gcc-4.8.2_src/host-x86_64-unknown-linux-gnu/gcc/ -B/data/gcc-4.8.2/x86_64-unknown-linux-gnu/bin/ -B/data/gcc-4.8.2/x86_64-unknown-linux-gnu/lib/ -isystem /data/gcc-4.8.2/x86_64-unknown-linux-gnu/include -isystem /data/gcc-4.8.2/x86_64-unknown-linux-gnu/sys-include   
checking for suffix of object files... configure: error: in `/data/gcc-4.8.2_src/x86_64-unknown-linux-gnu/libgcc':
configure: error: cannot compute suffix of object files: cannot compile
See `config.log' for more details.
make[2]: *** [configure-stage1-target-libgcc] 错误 1


所以只需要如下操作下即可：
export LD_LIBRARY_PATH=/usr/local/mpc/lib:/usr/local/mpfr/lib:/usr/local/gmp/lib:$LD_LIBRARY_PATH


注：gcc-4.8.2依赖mpc、mpfr和gmp：
./configure --prefix=/usr/local/gcc-4.8.2 --with-mpc=/usr/local/mpc-1.0.3 --with-mpfr=/usr/local/mpfr-3.1.3 --with-gmp=/usr/local/gmp-6.0.0


而mpc又依赖于：mpfr和gmp：
./configure --prefix=/usr/local/mpc-1.0.3 --with-mpfr=/usr/local/mpfr-3.1.3 --with-gmp=/usr/local/gmp-6.0.0


26） 编译gcc时，如果遇到下面这个错误：
fatal error: gnu/stubs-32.h: No such file or directory
这是因为在x86_64上，默认会编译出32位和64位两个版本。这样编译32位时，需要机器上有32位的libc头文件和库文件，但一些机器上可能没有，比如没有/lib目录，只有/lib64目录，这表示不支持32位的libc。为解决这个问题，可以禁止编译32位版本，在configure时带上参数--disable-multilib，或者安装32位版本的glibc。
 
27）某次编译遇到如下这样一个链接错误：
redis_dbi.cpp:224: undefined reference to `sdscatlen(char*, void const*, unsigned long)'
按常理，这个错误要么是没有指定相应的库，要么是静态库间的顺序问题。




但经过检查，这两个原因，而是因为gcc和g++混用原因：
1. 库libhiredis.a和libhiredis.so是由gcc编译出来的
2. 而调用它的代码是由g++编译的，因此导致了此问题。




问题的解决办法有两个：
1. 修改sdscatlen所在的.h文件，将代码使用
#ifdef __cplusplus
extern "C" {
#endif
修饰起来，或者直接使用“extern C”修饰函数sdscatlen。




2. 不修改redis的代码，在引用sds.h时加上“extern "C" {”：
extern "C" {
#include "sds.h"
}


上面两个办法均可，当然也可以考虑改用g++编译redis，不过可能会遇到很多错误。
redis对外供外部直接使用的头文件hiredis.h已使用了extern "C" {，所以不存在问题，只有当跳过hiredis.h，去使用一些内部头文件时需要注意一下。


28）x.hpp:27: error: expected identifier before string constant
x.hpp:27: error: expected `}' before string constant
x.hpp:27: error: expected unqualified-id before string constant


这个错误，可能是存在和枚举等同名的字符串宏，比如存在下面的宏定义：
enum DBTYPE
{
    UNDEFINE = 0,
    MYSQL_DB = 1,
    ORACLE_DB = 2
};


而另一.h文件中定义了宏：
#define MYSQL_DB "mysql"


29) 下面这个错误是因为类成员函数的声明和定义的返回值不相同
test.cpp:201:6: 错误：‘bool foo(const string&, const string&, const string&, const string&, const string&, const HBaseRowValue&)’的原型不匹配类‘CTest’中的任何一个
 bool CHBaseOper::foo(const std::string& tablename, const std::string& rowkey, const std::string& familyname, const std::string& columnname, const std::string& columnvalue, const HBaseRowValue& row_values)
      ^
In file included from test.cpp:8:0:
test.h:58:6: 错误：备选为：int foo(const string&, const string&, const string&, const string&, const string&, const HBaseRowValue&)
  int foo(const std::string& tablename, const std::string& rowkey, const std::string& familyname, const std::string& columnname, const std::string& columnvalue, const HBaseRowValue& row_values);


30)
messenger.cpp:5: error: expected unqualified-id before ':' token
该编译错误原因是：
CMessager:CMessager()
{
}
即少了个冒号：
CMessager::CMessager()
{
}


31) unable to find a register to spill in class ‘SIREG’
编译时如果遇到这个错误，表示遇到一个gcc的bug，最简单的办法是去掉编译参数中的-O3等优化项，然后再试可能就成功了，也可以考虑指定-fno-schedule-insns。


32）像下面这样一大堆乱七八糟的错误，可以考虑是否为少了“}”等引起的
/usr/include/c++/4.8.2/bits/stl_list.h:131:15: 错误：‘bidirectional_iterator_tag’不是命名空间‘gongyi::std’中的一个类型名
       typedef std::bidirectional_iterator_tag    iterator_category;
               ^
/usr/include/c++/4.8.2/bits/stl_list.h: 在成员函数‘_Tp* gongyi::std::_List_iterator<_tp>::operator->() const’中:
/usr/include/c++/4.8.2/bits/stl_list.h:150:16: 错误：‘__addressof’不是‘gongyi::std’的成员
       { return std::__addressof(static_cast<_Node*>(_M_node)->_M_data); } 
比如：
namespace A { namespace B {
    。。。。。。
// 下面少了一个“}”
} // namespace A { namespace B {
 

33）crc32’ cannot be used as a function
uint32_t crc32 = crc32(0L, (const unsigned char*)crc32_str.data(), crc32_str.size());
错误是因为函数名和变量名相同了，可以改成如下解决：
uint32_t crc32 = ::crc32(0L, (const unsigned char*)crc32_str.data(), crc32_str.size());

34) 
/data/X/mooon/tools/hbase_stress.cpp: In function 'void test_write(const std::map<std::basic_string, std::basic_string >&)':
/data/X/mooon/tools/hbase_stress.cpp:78:117: error: invalid conversion from 'void (*)(uint8_t, const string&, uint16_t) {aka void (*)(unsigned char, const std::basic_string&, short unsigned int)}' to 'mooon::sys::FunctionWith3Parameter<unsigned char,="" std::basic_string, short unsigned int>::FunctionPtr {aka void (*)(unsigned char, std::basic_string, short unsigned int)}' [-fpermissive]
             stress_threads[i] = new mooon::sys::CThreadEngine(mooon::sys::bind(write_thread, i, hbase_ip, hbase_port));                                                                                                                     ^
In file included from /data/X/mooon/tools/hbase_stress.cpp:24:0:
/data/X/mooon/tools/../include/mooon/sys/thread_engine.h:777:16: error:   initializing argument 1 of 'mooon::sys::Functor mooon::sys::bind(typename mooon::sys::FunctionWith3Parameter::FunctionPtr, Parameter1Type, Parameter2Type, Parameter3Type) [with Parameter1Type = unsigned char; Parameter2Type = std::basic_string; Parameter3Type = short unsigned int; typename mooon::sys::FunctionWith3Parameter::FunctionPtr = void (*)(unsigned char, std::basic_string, short unsigned int)]' [-fpermissive]
 inline Functor bind(typename FunctionWith3Parameter::FunctionPtr function_ptr, Parameter1Type parameter1, Parameter2Type parameter2, Parameter3Type parameter3)


上面这个错误的意思是第一个参数的类型为
void (*)(unsigned char, std::basic_string, short unsigned int)
但传入的类型为
void (*)(unsigned char, const std::basic_string&, short unsigned int)


从上面的对比可以看出，要求函数的第二个参数为std::string类型，而不是const std::string&类型


