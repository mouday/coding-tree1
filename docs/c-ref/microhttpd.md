# microhttpd

HTTP server

https://www.gnu.org/software/libmicrohttpd/

下载安装

```shell
# 下载
wget https://ftpmirror.gnu.org/libmicrohttpd/libmicrohttpd-latest.tar.gz

wget https://mirrors.aliyun.com/gnu/libmicrohttpd/libmicrohttpd-latest.tar.gz

# 解压
tar -zxvf libmicrohttpd-latest.tar.gz

# 安装
./configure --prefix=$HOME/local/libmicrohttpd-1.0.10 \
&& make && make install
```

环境变量 `~/.bash_profile`

```shell
# libmicrohttpd
MICROHTTPD_HOME=$HOME/local/libmicrohttpd-1.0.10

# 动态库
export LD_LIBRARY_PATH=$MICROHTTPD_HOME/lib:$LD_LIBRARY_PATH

# 静态库
export LIBRARY_PATH=$MICROHTTPD_HOME/lib:$LIBRARY_PATH

# C
export C_INCLUDE_PATH=$MICROHTTPD_HOME/include:$C_INCLUDE_PATH

# CPP
export CPLUS_INCLUDE_PATH=$MICROHTTPD_HOME/include:$CPLUS_INCLUDE_PATH
```

demo.c

```cpp
#include <microhttpd.h>
#include <stdlib.h>
#include <string.h>
#include <stdio.h>

#define PAGE "<html><head><title>libmicrohttpd demo</title>"\
             "</head><body>libmicrohttpd demo</body></html>"

static enum MHD_Result
ahc_echo(void * cls,
         struct MHD_Connection * connection,
         const char * url,
         const char * method,
         const char * version,
         const char * upload_data,
         size_t * upload_data_size,
         void ** ptr) {
  static int dummy;
  const char *page = (const char *) cls;
  struct MHD_Response *response;
  enum MHD_Result ret;

  if (0 != strcmp(method, "GET"))
    return MHD_NO; /* unexpected method */
  if (&dummy != *ptr)
  {
    /* The first time only the headers are valid,
       do not respond in the first round... */
    *ptr = &dummy;
    return MHD_YES;
  }
  if (0 != *upload_data_size)
    return MHD_NO; /* upload data in a GET!? */
  *ptr = NULL; /* clear context pointer */
  response = MHD_create_response_from_buffer (strlen(page),
                                              (void*) page,
                          MHD_RESPMEM_PERSISTENT);
  ret = MHD_queue_response(connection,
               MHD_HTTP_OK,
               response);
  MHD_destroy_response(response);
  return ret;
}

int
main(int argc,
     char ** argv) {
  struct MHD_Daemon * d;

  if (argc != 2)
  {
    printf("%s PORT\n",
       argv[0]);
    return 1;
  }
  d = MHD_start_daemon(MHD_USE_THREAD_PER_CONNECTION,
               atoi(argv[1]),
               NULL,
               NULL,
               &ahc_echo,
               PAGE,
               MHD_OPTION_END);
  if (NULL == d)
    return 1;
  (void) getc (stdin);
  MHD_stop_daemon(d);
  return 0;
}
```

编译启动

```shell
$ gcc demo.c -lmicrohttpd

./a.out 8000
```

访问

```shell
$ curl http://127.0.0.1:8000/
<html><head><title>libmicrohttpd demo</title></head><body>libmicrohttpd demo</body></html>% 
```

源码

```shell
$ git clone https://git.gnunet.org/libmicrohttpd.git
```
