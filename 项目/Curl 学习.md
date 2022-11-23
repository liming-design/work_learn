
```c
//https://www.52dixiaowo.com/post-3252.html#article_nav_3
#include <stdio.h>
#include <stdlib.h>

#include <curl/curl.h>

int main(void)
{
    // 1.创建一个curl句柄
    CURL *curl = NULL;
    CURLcode res;
    
    // 2.初始化一个curl句柄
    curl = curl_easy_init();
    
    // 3.给句柄设置一些参数
    curl_easy_setopt(curl, CURLOPT_URL, "http://www.baidu.com");
    	
    // 4.将curl句柄向远程服务器提交
    res = curl_easy_perform(curl);
    if( res != CURLE_OK){
        printf("curl easy perform error res = %d",res);
        return 1;
    }
    // 5.初始化服务器的响应结果
    // 6.释放句柄空间
    curl_easy_cleanup(curl);
}

```

