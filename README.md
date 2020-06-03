### 内联汇编
内联汇编是C/C++的一个重要特性，内联汇编顾名思义是可以在C/C++ 语法内嵌套使用汇编指令，使用内联汇编的好处就是可以更灵活便捷，我觉得尤其是在做逆向这一块，简直不要太爽。

内联汇编Demo：

```
#include <stdio.h>
#include <windows.h>


int asmFunc()
{
	int a = 0;
	__asm
	{
		// 变量a 地址复制给eax
		mov eax, a
		// eax 地址做加点运算
		add eax, 0x1213
		// 将计算后的值传递a
		mov a, eax
	}
	// 返回eax
	return a;

}

int main()
{
	printf("0x%X\r\n", asmFunc());
	system("pause");

}

```
运行结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200603152313368.png)

以上是32 位C/C++ 使用内联汇编的小Demo，在x32 开发中，使用__asm 关键字就可以在其代码块内嵌入x32 汇编代码，好不方便。但是突然发现，x64 开发中，不支持这种内联汇编，尴尴尬尬，那我要是逆向一个x64 程序的时候怎么调用CALL（其实可以使用函数指针），而且也不灵活，不方便呀。好在有属于x64 的“内联汇编”

#### x64 内联汇编
1. 新建一个x64 控制台项目，
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020060315530712.png)
2. 生成自定义
	操作：项目名称（不是解决方案名称）右键---> 生成依赖项 ----> 生成自定义
	![在这里插入图片描述](https://img-blog.csdnimg.cn/20200603155545781.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTM1MDk3,size_16,color_FFFFFF,t_70)
3. 创建汇编文件（.Asm）并修改属性
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200603155832971.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTM1MDk3,size_16,color_FFFFFF,t_70)
在.asm 文件上右击，属性，设置如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200603155933200.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTM1MDk3,size_16,color_FFFFFF,t_70)
4. 接下来，可以写入汇编指令
	注：
	> 汇编格式
	```
	.CODE  ;这是开始
	
		; 这是注释（; + 注释内容）
	
		; 这是方法名称
		funcName PROC
			;这里写入汇编指令
		
	
		funcName ENDP
	
	
	
	END ;这是结束
	```
	> x64 汇编传递参数
	64 位程序传递参数仅使用__fastcall 约定，及前4个参数由寄存器rcx, rdx, r8, r9传递，其他使用堆栈传递。[详细](https://blog.csdn.net/cosmoslife/article/details/8771824)；[fastcall](https://baike.baidu.com/item/__fastcall/3736920?fr=aladdin)
	![在这里插入图片描述](https://img-blog.csdnimg.cn/20200603161940726.png)
	
	>与高级语言开发类似，函数名称不要使用指令名称，如add, test, mov

##### 使用x64 汇编实现add(int a, int b, int c, int d, int e, int f) 函数
###### .asm

```
.CODE
myAdd PROC
	sub rsp, 20h
	xor rax, rax
	add rax, rcx
	add rax, rdx
	add rax, r8	
	add rax, r9	
	add rax, [rsp + 50h]
	add rax, [rsp + 48h]

	add rsp, 20h
	ret
myAdd ENDP

END 
```
###### .cpp

```
#include <windows.h>
#include <stdio.h>

// 使用C 的方式
EXTERN_C UINT64 myAdd(UINT64 arg1, UINT64 arg2, UINT64 arg3, UINT64 arg4, UINT64 arg5, UINT64 arg6);


int main()
{
	Sleep(1);
	printf("%d\r\n", myAdd(1, 1, 1, 1, 1, 1));
	Sleep(1);
	system("pause");
}
```

##### 升级，使用汇编调用call
> 这里我说一下我的思路，之前x64 程序我调用call 都是使用函数指针来调用，这种方式虽然可以很方便的调用call,但是无奈不灵活（也可能是因为我太太太太太太太太太太菜）只能去使用已有的功能，所以这次使用汇编调用call，实现功能的修改，以下所有程序都是自写测试用：

项目大概：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200603165653502.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTM1MDk3,size_16,color_FFFFFF,t_70)
运行效果：
运行前：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200603165814276.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTM1MDk3,size_16,color_FFFFFF,t_70)
运行后：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200603170003297.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwNTM1MDk3,size_16,color_FFFFFF,t_70)

###### 源程序：
###### 1. 目标程序
```
#include <iostream>
#include <stdio.h>
#include <windows.h>

int add(int a, int b)
{
	printf("test Msg\r\n");
	return a + b;
}

int main()
{
	Sleep(1);
	printf("结果 = %d\r\n", add(1, 555));
	Sleep(1);
	system("pause");
	return 0;
}
```

###### 2. 注入程序
```
#include <windows.h>
#include <stdio.h>


#define DLL_PATH "D:\\c_work\\x64远程调用CALL_汇编测试\\x64\\bin\\Dll.dll"
#define TAR_PROCESS_CLASS "ConsoleWindowClass"
#define TAR_PROCESS_TITLE "D:\\c_work\\x64远程调用CALL_汇编测试\\x64\\bin\\TarProject.exe"

int main()
{
	// 获取PID
	DWORD pId = 0;
	HWND hWnd = FindWindowA(TAR_PROCESS_CLASS, TAR_PROCESS_TITLE);
	GetWindowThreadProcessId(hWnd, &pId);

	// 打开进程
	HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, pId);

	// 开辟内存
	LPWORD lpAddr = (LPWORD)VirtualAllocEx(hProcess, NULL, 250, MEM_COMMIT, PAGE_READWRITE);

	// 写入DLL 全路径
	char path[256] = { 0 };
	strcpy(path, DLL_PATH);
	WriteProcessMemory(hProcess, lpAddr, path, strlen(path) + 1, 0);

	CreateRemoteThread(hProcess,
		NULL,
		NULL,
		(LPTHREAD_START_ROUTINE)LoadLibraryA,
		lpAddr,
		NULL,
		NULL);


}

```

###### 3. 待注入dll
_3.1. ASM_

```
.code


testAsmFunc PROC

;000000014000 | BA 2B020000        | mov edx,22B                            | maintest.cpp:14
;000000014000 | B9 01000000        | mov ecx,1                              |
;000000014000 | E8 B2FFFFFF        | call <tarproject.int __cdecl add(int,i | call 0x00000001400010E0

	sub rsp, 50h
	mov edx, 22h
	mov ecx, 1h
	mov rax, 1400010E0h
	call rax


	add rsp, 50h
	ret


testAsmFunc ENDP


end
```
_3.2 CPP_

```
// dllmain.cpp : 定义 DLL 应用程序的入口点。
#include "pch.h"
#include "stdio.h"
#include "windows.h"


extern "C" UINT64 testFunc(int a, int b);
extern "C" UINT64 testAsmFunc();

BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
	{
		// 加载
		int value = testAsmFunc();
		char szpValue[256] = { 0 };
		sprintf(szpValue, "%d", value);
		MessageBox(NULL, szpValue, szpValue, MB_OK);

	}; break;
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}


```
