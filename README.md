Using Intel’s Secure Key (RDRAND) in MS Visual C++ 2013
-------------------------------------------------------

このプロジェクトは  
[https://github.com/viathefalcon/rdrand_msvc_2010](https://github.com/viathefalcon/rdrand_msvc_2010)  
のforkで、Visual Studio 2013 Community向けです。それ以前のVCにはfork元のプロジェクトを使用してください。  
またgccコンパイラーには  
[Ivy BridgeのRDRAND命令はモンテカルロ法に使えるか ( パソコン ) - 正統納豆天国ブログ - Yahoo!ブログ](http://blogs.yahoo.co.jp/natto_heaven/32698902.html)  
を参照してほしい。

Intel Core iシリーズの第3世代より搭載されたRDRAND命令を利用して乱数を得るプログラムになっている。  
RDRAND命令やハードウェア乱数生成器そのものについては  
[ハードウェア乱数生成器は信頼できるか | 本の虫](http://cpplover.blogspot.jp/2013/07/blog-post_14.html)  
の解説が詳しい。  
そのRDRAND命令だが、環境によっては ``immintrin.h`` をincludeすることで

```cpp
int _rdrand_u16_step(unsigned short *);  
int _rdrand_u32_step(unsigned int *);  
int _rdrand_u64_step(unsigned long long *); 
```

これらの関数が利用可能になるが、  
[C++ - ハードウェア乱数 RDRAND命令の使い方 - Qiita](http://qiita.com/Seizh/items/3e55b04e62d66808fd03)  
VisualStudio2013Communityでは

```cpp
static bool RDRAND(void) { return CPU_Rep.f_1_ECX_[30]; }
extern int __cdecl _rdrand16_step(unsigned short *);
extern int __cdecl _rdrand32_step(unsigned int *);
#if defined (_M_X64)
extern int __cdecl _rdrand64_step(unsigned __int64 *);
#endif  /* defined (_M_X64) */
```
を利用するらしい。(さすがMSVC、IntelのReferenceと関数名を揃える気なんてないんやで・・・。おのれMSVC。)  
[__cpuid, __cpuidex | MSDN](https://msdn.microsoft.com/ja-jp/library/hskdteyh.aspx)  
しかし、その実装を見るのはそれなりに有意義だと思う。  
ところがVisualStudio2013ではx64buildではインラインアセンブリすら使えないので別途.asmファイルを作成する必要がある。  

ia_rdrand.hによって追加されるのは

```cpp
bool rdrand_supported(void);//RDRND命令をサポートするかどうか。不明:2,対応:1,不対応:0
bool rdrand(unsigned int* dest);//RDRND命令による乱数生成。成功時はtrueをかえす。乱数は変数destに入る
```

但し、rdrand関数はソリューションプラットフォームをWin32にした場合は``_rdrand_u32_step``関数のように振る舞い、x64にした場合は``_rdrand_u64_step``関数のように振る舞う。

下記にもあるようにこのソースコードの大部分は  
[Intel® Digital Random Number Generator (DRNG) Software Implementation Guide](https://software.intel.com/en-us/articles/intel-digital-random-number-generator-drng-software-implementation-guide)  
を参考に書かれている。  
下にfork元のreadmeを載せておく。

--[yumetodo](https://twitter.com/yumetodo)

-------------------------------------------------------

Among the features added to Intel’s 3rd-Generation Core i* processors is a Digital Random Number Generator (DRNG) backed by an on-die hardware entropy source. This new hardware feature is made available to software via the also-new RDRAND instruction.

If you’re still using the compiler which shipped with Visual C++ 2010, it seems the only way to leverage the DRNG is either via a third-party library (the one available from Intel’s website was, as of writing, broken) or by dipping into assembly programming. Of these the latter comes with a couple of catches: the mnenomic/intrinsic for the instruction is not available for older assemblers/compilers, and the assembly is slightly different for 32- and 64-bit environments.

This project demonstrates testing whether the host processor supports the RDRAND instruction as well as invoking it (via assembly). When built for 32-bit CPUs, the assembly is inlined; when built for 64-bit CPUs, the assembly is linked in via an exernal module (the 64-bit compiler in Visual Studio 2010 does not support inline assembly).

For the most part, the project simply follows the Software Implementation Guide from Intel. Additionally, it demonstrates invoking the instruction via its opcode, and linking a module implemented in assembly into a VC++ project.
