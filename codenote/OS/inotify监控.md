---

title: inotify监控
date: 2019-2-26 18:53:12
categories: codenote
tags: [OS, Linux]

typora-copy-images-to: inotify监控
---

### `inotify`的API：

Inotify 提供一个简单的 API，使用最小的文件描述符，并且允许细粒度监控。与 inotify 的通信是通过系统调用实现。可用的函数如下所示：

- inotify_init
  是用于创建一个 inotify 实例的系统调用，并返回一个指向该实例的文件描述符。

- inotify_init1
  与 inotify_init 相似，并带有附加标志。如果这些附加标志没有指定，将采用与 inotify_init 相同的值。

- inotify_add_watch
  增加对文件或者目录的监控，并指定需要监控哪些事件。标志用于控制是否将事件添加到已有的监控中，是否只有路径代表一个目录才进行监控，是否要追踪符号链接，是否进行一次性监控，当首次事件出现后就停止监控。

- inotify_rm_watch
  从监控列表中移出监控项目。

- read
  读取包含一个或者多个事件信息的缓存。

- close
  关闭文件描述符，并且移除所有在该描述符上的所有监控。当关于某实例的所有文件描述符都关闭时，资源和下层对象都将释放，以供内核再次使用。

因此，典型的监控程序需要进行如下操作：

- 使用 inotify_init 打开一个文件描述符
- 添加一个或者多个监控
- 等待事件
- 处理事件，然后返回并等待更多事件
- 当监控不再活动时，或者接到某个信号之后，关闭文件描述符，清空，然后退出。

### 事件

事件的通过顺序的方式被读取到缓存当中。

用于`inotify`中的事件结构体

```c
struct inotify_event
{
  int wd;               /* Watch descriptor.  */
  uint32_t mask;        /* Watch mask.  */
  uint32_t cookie;      /* Cookie to synchronize two events.  */
  uint32_t len;         /* Length (including NULs) of name.  */
  char name __flexarr;  /* Name.  */
  };
```

### 可监控的事件类型

有几种事件能够被监控。一些事件，比如 IN_DELETE_SELF 只适用于正在被监控的项目，而另一些，比如 IN_ATTRIB 或者 IN_OPEN 则只适用于监控过的项目，或者如果该项目是目录，则可以应用到其所包含的目录或文件。

- IN_ACCESS
  被监控项目或者被监控目录中的条目被访问过。例如，一个打开的文件被读取。
- IN_MODIFY
  被监控项目或者被监控目录中的条目被修改过。例如，一个打开的文件被修改。
- IN_ATTRIB
  被监控项目或者被监控目录中条目的元数据被修改过。例如，时间戳或者许可被修改。
- IN_CLOSE_WRITE
  一个打开的，等待写入的文件或目录被关闭。
- IN_CLOSE_NOWRITE
  一个以只读方式打开的文件或目录被关闭。
- IN_CLOSE
  一个掩码，可以很便捷地对前面提到的两个关闭事件（IN_CLOSE_WRITE | IN_CLOSE_NOWRITE）进行逻辑操作。
- IN_OPEN
  文件或目录被打开。
- IN_MOVED_FROM
  被监控项目或者被监控目录中的条目被移出监控区域。该事件还包含一个 cookie 来实现 IN_MOVED_FROM 与 IN_MOVED_TO 的关联。
- IN_MOVED_TO
  文件或目录被移入监控区域。该事件包含一个针对 IN_MOVED_FROM 的 cookie。如果文件或目录只是被重命名，将能看到这两个事件，如果它只是被移入或移出非监控区域，将只能看到一个事件。如果移动或重命名一个被监控项目，监控将继续进行。参见下面的 IN_MOVE-SELF。
- IN_MOVE
  可以很便捷地对前面提到的两个移动事件（IN_MOVED_FROM | IN_MOVED_TO）进行逻辑操作的掩码。
- IN_CREATE
  在被监控目录中创建了子目录或文件。
- IN_DELETE
  被监控目录中有子目录或文件被删除。
- IN_DELETE_SELF
  被监控项目本身被删除。监控终止，并且将收到一个 IN_IGNORED 事件。
- IN_MOVE_SELF
  监控项目本身被移动。
  除了事件标志以外，还可以在 `inotify` 头文件（`/usr/include/sys/inotify.h`）中找到其他几个标志。例如，如果只想监控第一个事件，可以在增加监控时设置 `IN_ONESHOT `标志。

### 示例

最简单的例子，可监视到某文件被操作，但由于事件的发生方式比较复杂，因此以下例子只作最简单的示例用

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/inotify.h>
#include <sys/select.h>
#include <unistd.h>

int main(int argc, char **argv)
{
    
    int inofd;
    inofd = inotify_init();
    if (inofd < 0) {

        printf("inotify_init\n");

    }
    
    if(inotify_add_watch(inofd, "/home/amlloc/workspace/study/inotify-sample/mysample/text.txt", IN_ALL_EVENTS) < 0){
        printf("add inotify failed");
    }

    
    while (1)
    {
        fd_set fd_s;
        FD_ZERO(&fd_s);
        FD_SET(inofd, &fd_s);
        if (select(FD_SETSIZE, &fd_s, NULL, NULL, NULL) >= 0)
        {
            printf("asdasdasd");
        }else
        {
            printf("XXXXXXXX");
        }

    }
}

```

一个简单的官方例子，这个例子可以较为全面地解决事件排队问题，具体解释直接看代码注释吧~

- inotify_test

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/ioctl.h>
#include <sys/select.h>
#include <sys/inotify.h>
#include <signal.h>

#include "event_queue.h"
#include "inotify_utils.h"
int keep_running;

/* This program will take as arguments one or more directory 
   or file names, and monitor them, printing event notifications 
   to the console. It will automatically terminate if all watched
   items are deleted or unmounted. Use ctrl-C or kill to 
   terminate otherwise.
*/

/* Signal handler that simply resets a flag to cause termination */
void signal_handler(int signum)
{
  keep_running = 0;
}

int main(int argc, char **argv)
{
  /* This is the file descriptor for the inotify watch */
  int inotify_fd;

  keep_running = 1;

  /* Set a ctrl-c signal handler */
  /**
   * signal函数返回值是指向之前的信号处理程序的指针。
   */ 
  if (signal(SIGINT, signal_handler) == SIG_IGN)
  {
    /* Reset to SIG_IGN (ignore) if that was the prior state */
    signal(SIGINT, SIG_IGN);
  }

  /* First we open the inotify dev entry */
  inotify_fd = open_inotify_fd();
  if (inotify_fd > 0)
  {

    /* We will need a place to enqueue inotify events,
         this is needed because if you do not read events
         fast enough, you will miss them. This queue is 
         probably too small if you are monitoring something
         like a directory with a lot of files and the directory 
         is deleted.
       */
    queue_t q;
    q = queue_create(128);

    /* This is the watch descriptor returned for each item we are 
         watching. A real application might keep these for some use 
         in the application. This sample only makes sure that none of
         the watch descriptors is less than 0.
       */
    int wd;

    /* Watch all events (IN_ALL_EVENTS) for the directories and 
         files passed in as arguments.
         Read the article for why you might want to alter this for 
         more efficient inotify use in your app.      
       */
    int index;
    wd = 0;
    printf("\n");
    for (index = 1; (index < argc) && (wd >= 0); index++)
    {
      wd = watch_dir(inotify_fd, argv[index], IN_ALL_EVENTS);
      /*wd = watch_dir (inotify_fd, argv[index], IN_ALL_EVENTS & ~(IN_CLOSE | IN_OPEN) ); */
    }

    if (wd > 0)
    {
      /* Wait for events and process them until a 
             termination condition is detected
           */
      process_inotify_events(q, inotify_fd);
    }
    printf("\nTerminating\n");

    /* Finish up by closing the fd, destroying the queue,
         and returning a proper code
       */
    close_inotify_fd(inotify_fd);
    queue_destroy(q);
  }
  return 0;
}

```

- `inotyfy_utils.c`

  ```c
  #include <stdio.h>
  #include <stdlib.h>
  #include <stddef.h>
  #include <string.h>
  #include <sys/types.h>
  #include <unistd.h>
  #include <sys/select.h>
  
  #include <sys/inotify.h>
  
  #include "event_queue.h"
  #include "inotify_utils.h"
  
  extern int keep_running;
  static int watched_items;
  
  /* Create an inotify instance and open a file descriptor
     to access it */
  int open_inotify_fd()
  {
    int fd;
  
    watched_items = 0;
    fd = inotify_init();
  
    if (fd < 0)
    {
      perror("inotify_init () = ");
    }
    return fd;
  }
  
  /* Close the open file descriptor that was opened with inotify_init() */
  int close_inotify_fd(int fd)
  {
    int r;
  
    if ((r = close(fd)) < 0)
    {
      perror("close (fd) = ");
    }
  
    watched_items = 0;
    return r;
  }
  
  /* This method does the work of determining what happened,
     then allows us to act appropriately
     @param event 被处理的事件
  */
  void handle_event(queue_entry_t event)
  {
    /* If the event was associated with a filename, we will store it here */
    char *cur_event_filename = NULL;
    char *cur_event_file_or_dir = NULL;
    /* This is the watch descriptor the event occurred on */
    int cur_event_wd = event->inot_ev.wd;
    int cur_event_cookie = event->inot_ev.cookie;
    unsigned long flags;
  
    if (event->inot_ev.len)
    {
      cur_event_filename = event->inot_ev.name;
    }
    if (event->inot_ev.mask & IN_ISDIR)
    {
      cur_event_file_or_dir = "Dir";
    }
    else
    {
      cur_event_file_or_dir = "File";
    }
    flags = event->inot_ev.mask &
            ~(IN_ALL_EVENTS | IN_UNMOUNT | IN_Q_OVERFLOW | IN_IGNORED);
  
    /* Perform event dependent handler routines */
    /* The mask is the magic that tells us what file operation occurred */
    switch (event->inot_ev.mask &
            (IN_ALL_EVENTS | IN_UNMOUNT | IN_Q_OVERFLOW | IN_IGNORED))
    {
      /* File was accessed */
    case IN_ACCESS:
      printf("ACCESS: %s \"%s\" on WD #%i\n",
             cur_event_file_or_dir, cur_event_filename, cur_event_wd);
      break;
  
      /* File was modified */
    case IN_MODIFY:
      printf("MODIFY: %s \"%s\" on WD #%i\n",
             cur_event_file_or_dir, cur_event_filename, cur_event_wd);
      break;
  
      /* File changed attributes */
    case IN_ATTRIB:
      printf("ATTRIB: %s \"%s\" on WD #%i\n",
             cur_event_file_or_dir, cur_event_filename, cur_event_wd);
      break;
  
      /* File open for writing was closed */
    case IN_CLOSE_WRITE:
      printf("CLOSE_WRITE: %s \"%s\" on WD #%i\n",
             cur_event_file_or_dir, cur_event_filename, cur_event_wd);
      break;
  
      /* File open read-only was closed */
    case IN_CLOSE_NOWRITE:
      printf("CLOSE_NOWRITE: %s \"%s\" on WD #%i\n",
             cur_event_file_or_dir, cur_event_filename, cur_event_wd);
      break;
  
      /* File was opened */
    case IN_OPEN:
      printf("OPEN: %s \"%s\" on WD #%i\n",
             cur_event_file_or_dir, cur_event_filename, cur_event_wd);
      break;
  
      /* File was moved from X */
    case IN_MOVED_FROM:
      printf("MOVED_FROM: %s \"%s\" on WD #%i. Cookie=%d\n",
             cur_event_file_or_dir, cur_event_filename, cur_event_wd,
             cur_event_cookie);
      break;
  
      /* File was moved to X */
    case IN_MOVED_TO:
      printf("MOVED_TO: %s \"%s\" on WD #%i. Cookie=%d\n",
             cur_event_file_or_dir, cur_event_filename, cur_event_wd,
             cur_event_cookie);
      break;
  
      /* Subdir or file was deleted */
    case IN_DELETE:
      printf("DELETE: %s \"%s\" on WD #%i\n",
             cur_event_file_or_dir, cur_event_filename, cur_event_wd);
      break;
  
      /* Subdir or file was created */
    case IN_CREATE:
      printf("CREATE: %s \"%s\" on WD #%i\n",
             cur_event_file_or_dir, cur_event_filename, cur_event_wd);
      break;
  
      /* Watched entry was deleted */
    case IN_DELETE_SELF:
      printf("DELETE_SELF: %s \"%s\" on WD #%i\n",
             cur_event_file_or_dir, cur_event_filename, cur_event_wd);
      break;
  
      /* Watched entry was moved */
    case IN_MOVE_SELF:
      printf("MOVE_SELF: %s \"%s\" on WD #%i\n",
             cur_event_file_or_dir, cur_event_filename, cur_event_wd);
      break;
  
      /* Backing FS was unmounted */
    case IN_UNMOUNT:
      printf("UNMOUNT: %s \"%s\" on WD #%i\n",
             cur_event_file_or_dir, cur_event_filename, cur_event_wd);
      break;
  
      /* Too many FS events were received without reading them
           some event notifications were potentially lost.  */
    case IN_Q_OVERFLOW:
      printf("Warning: AN OVERFLOW EVENT OCCURRED: \n");
      break;
  
      /* Watch was removed explicitly by inotify_rm_watch or automatically
           because file was deleted, or file system was unmounted.  */
    case IN_IGNORED:
      watched_items--;
      printf("IGNORED: WD #%d\n", cur_event_wd);
      printf("Watching = %d items\n", watched_items);
      break;
  
      /* Some unknown message received */
    default:
      printf("UNKNOWN EVENT \"%X\" OCCURRED for file \"%s\" on WD #%i\n",
             event->inot_ev.mask, cur_event_filename, cur_event_wd);
      break;
    }
    /* If any flags were set other than IN_ISDIR, report the flags */
    if (flags & (~IN_ISDIR))
    {
      flags = event->inot_ev.mask;
      printf("Flags=%lX\n", flags);
    }
  }
  
  /**
   * 处理事件队列
   * @param q 事件队列
   */
  void handle_events(queue_t q)
  {
    queue_entry_t event;
    while (!queue_empty(q))
    {
      event = queue_dequeue(q);
      handle_event(event);
      free(event);
    }
  }
  
  /**
   * 将存储在结构体q中，并排队
   * @param q 事件结构体队列
   * @param fd  inotify描述符
   * @return 此次进行排队的事件数量 
   */
  int read_events(queue_t q, int fd)
  {
    char buffer[16384];
    size_t buffer_i;
    struct inotify_event *pevent;
    queue_entry_t event;
    ssize_t r;
    size_t event_size, q_event_size;
    int count = 0;
  
    r = read(fd, buffer, 16384);
    if (r <= 0)
      return r;
    buffer_i = 0;
    while (buffer_i < r)
    {
      /* Parse events and queue them. */
      pevent = (struct inotify_event *)&buffer[buffer_i];
      event_size = offsetof(struct inotify_event, name) + pevent->len;
      q_event_size = offsetof(struct queue_entry, inot_ev.name) + pevent->len;
      event = malloc(q_event_size);
      memmove(&(event->inot_ev), pevent, event_size);
      queue_enqueue(event, q);
      buffer_i += event_size;
      count++;
    }
    printf("\n%d events queued\n", count);
    return count;
  }
  
  // 等待事件或者中断
  int event_check(int fd)
  {
    fd_set rfds;
    //将指定的fd_set清零
    FD_ZERO(&rfds);
    //然后调用宏FD_SET将需要测试的fd加入fd_set
    FD_SET(fd, &rfds);
    /* Wait until an event happens or we get interrupted 
       by a signal that we catch */
    //调用函数select测试fd_set中的所有fd
  
    //select 的用法
    /**
     * 可是使用Select就可以完成非阻塞（所谓非阻塞方式non-block，就是进程或线程执行此函数时不必非要等待事件的发生，一旦执行肯定返回，
     * 以返回值的不同来反映函数的执行情况，如果事件发生则与阻塞方式相同，
     * 若事件没有发生，则返回一个代码来告知事件未发生，而进程或线程继续执行，所以效率较高）方式工作的程序，
     * 它能够监视我们需要监视的文件描述符的变化情况——读写或是异常。
       返回值：准备就绪的描述符数，若超时则返回0，若出错则返回-1。
     * 
     * 使用select函数的过程一般是：
      先调用宏FD_ZERO将指定的fd_set清零，
      然后调用宏FD_SET将需要测试的fd加入fd_set，
      接着调用函数select测试fd_set中的所有fd，
      最后用宏FD_ISSET检查某个fd在函数select调用后，相应位是否仍然为1。 
     */
    return select(FD_SETSIZE, &rfds, NULL, NULL, NULL);
  }
  
  /**
   *  对事件进行处理的循环
   *  @param q  存储事件的数据结构  
   *  @param fd  inotify描述符
   *  @return 0
   */
  int process_inotify_events(queue_t q, int fd)
  {
    while (keep_running && (watched_items > 0))
    {
      if (event_check(fd) > 0)
      { 
        //有事件发生
        int r;
        r = read_events(q, fd);
        if (r < 0)
        {
          break;
        }
        else
        {
          handle_events(q);
        }
      }
    }
    return 0;
  }
  
  int watch_dir(int fd, const char *dirname, unsigned long mask)
  {
    int wd;
    wd = inotify_add_watch(fd, dirname, mask);
    if (wd < 0)
    {
      printf("Cannot add watch for \"%s\" with event mask %lX", dirname,
             mask);
      fflush(stdout);
      perror(" ");
    }
    else
    {
      watched_items++;
      printf("Watching %s WD=%d\n", dirname, wd);
      printf("Watching = %d items\n", watched_items);
    }
    return wd;
  }
  
  int ignore_wd(int fd, int wd)
  {
    int r;
    r = inotify_rm_watch(fd, wd);
    if (r < 0)
    {
      perror("inotify_rm_watch(fd, wd) = ");
    }
    else
    {
      watched_items--;
    }
    return r;
  }
  
  ```

- event_queue.c

  ```c
  #include "event_queue.h"
  #include <stdlib.h>
  #include <stdint.h>
  
  /* Simple queue implemented as a singly linked list with head 
     and tail pointers maintained
  */
  int queue_empty (queue_t q)
  {
    return q->head == NULL;
  }
  
  queue_t queue_create ()
  {
    queue_t q;
    q = malloc (sizeof (struct queue_struct));
    if (q == NULL)
      exit (-1);
  
    q->head = q->tail = NULL;
    return q;
  }
  
  void queue_destroy (queue_t q)
  {
    if (q != NULL)
      {
        while (q->head != NULL)
  	{
  	  queue_entry_t next = q->head;
  	  q->head = next->next_ptr;
  	  next->next_ptr = NULL;
  	  free (next);
  	}
        q->head = q->tail = NULL;
        free (q);
      }
  }
  
  void queue_enqueue (queue_entry_t d, queue_t q)
  {
    d->next_ptr = NULL;
    if (q->tail)
      {
         q->tail->next_ptr = d;
         q->tail = d;
      }
    else
      {
        q->head = q->tail = d;
      }
  }
  
  queue_entry_t  queue_dequeue (queue_t q)
  {
    queue_entry_t first = q->head;
    if (first)
      {
        q->head = first->next_ptr;
        if (q->head == NULL) 
  	{
  	  q->tail = NULL;
  	}
        first->next_ptr = NULL;
      }
    return first;
  }
  
  ```

  需要代码可以直接在下面我学习的链接中进行下载

  [参考文献](https://www.ibm.com/developerworks/cn/linux/l-inotify/index.html?ca=drs-)