---
title: Android Easy Sample
date: 2018-10-10 21:16:01
categories: codenote
tags: Android
---

Android 常用简单的测试性模板代码积累
<!--more-->

## Android 用于功能测试界面的例子

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
              android:orientation="vertical">

    <TextView
        android:id="@+id/Title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentTop="true"
        android:layout_centerInParent="true"
        android:text="test test sample"
        android:textSize="35sp"/>



    <Button
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:onClick="onClickTest"
        android:text="test1"/>

    <Button
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:onClick="onClickThread"
        android:text="多线程不断调用"/>

    <TextView
        android:id="@+id/token"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_below="@+id/Title"
        android:layout_centerHorizontal="true"
        android:paddingTop="5dp"
        android:text="(空)"
        android:textSize="18sp"/>



</LinearLayout>
```

## 打印堆栈调用的例子

- 必要的代码(没有深究)

```c++
#include <unwind.h>
#include <fcntl.h>

struct BacktraceState
{
    intptr_t* current;
    intptr_t* end;
};


static _Unwind_Reason_Code unwindCallback(struct _Unwind_Context* context, void* arg)
{
    BacktraceState* state = static_cast<BacktraceState*>(arg);
    intptr_t ip = (intptr_t)_Unwind_GetIP(context);
    if (ip) {
        if (state->current == state->end) {
            return _URC_END_OF_STACK;
        } else {
            state->current[0] = ip;
            state->current++;
        }
    }
    return _URC_NO_REASON;


}

size_t captureBacktrace(intptr_t* buffer, size_t maxStackDeep)
{
    BacktraceState state = {buffer, buffer + maxStackDeep};
    _Unwind_Backtrace(unwindCallback, &state);
    return state.current - buffer;
}

void dumpBacktraceIndex(char *out, intptr_t* buffer, size_t count)
{
    for (size_t idx = 0; idx < count; ++idx) {
        intptr_t addr = buffer[idx];
        const char* symbol = "      ";
        const char* dlfile="      ";

        Dl_info info;
        if (dladdr((void*)addr, &info)) {
            if(info.dli_sname){
                symbol = info.dli_sname;
            }
            if(info.dli_fname){
                dlfile = info.dli_fname;
            }
        }else{
            strcat(out,"#                               \n");
            continue;
        }
        char temp[50];
        memset(temp,0,sizeof(temp));
        sprintf(temp,"%zu",idx);
        strcat(out,"#");
        strcat(out,temp);
        strcat(out, ": ");
        memset(temp,0,sizeof(temp));
        sprintf(temp,"%x",addr);
        strcat(out,temp);
        strcat(out, "  " );
        strcat(out, symbol);
        strcat(out, "      ");
        strcat(out, dlfile);
        strcat(out, "\n" );
    }
}



void TraceBuffer(const char* tag, const void* buffer, size_t size)
{
    std::string myLog = "";
    if (buffer && size > 0)
    {
        char temp[3] = {0};
        for (size_t i = 0; i < size; i++) {

            uint8_t* p = (uint8_t* )buffer;
            sprintf(temp, "%.2hhX", p[i]);

            if (i == size - 1)
            {
                myLog += temp;
            }
            else
            {
                myLog += temp;
                myLog += " ";
            }
        }
        LOGE("[%s], len=%zu/0x%zX, %s", tag, size, size, myLog.c_str());
    }
}
```

- 然后在需要打印堆栈调用的地方放置如下代码 

```c++
const size_t maxStackDeep = 20;
intptr_t stackBuf[maxStackDeep];
char outBuf[2048];
memset(outBuf,0,sizeof(outBuf));
dumpBacktraceIndex(outBuf, stackBuf, captureBacktrace(stackBuf, maxStackDeep));
LOGE(" %s\n", outBuf);
```

