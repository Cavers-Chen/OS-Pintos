# PINTOS PROJECT

**2020-2021**

**供学弟学妹们参考**

## 前言

* 预备的基础知识： C、Linux(基本的界面操作以及命令行语句)
* 结合本报告学习的用时估计 hw2-三到四天 scheduling-一周 userprog-一周（这一部分建议再看一下别的参考资料，特别是四位对其和压栈的逻辑） 

这是我们学校2021年秋季的操作系统课程设计，使用的是伯克利CS162的Pintos框架。今年的pintos和往年相比有很大的变化，往年的project2有80个测试点，今年加到了96个，这是因为增加了FPU内容。另外，市面上的Pintos手册的版本也属于层次不齐，所以如果你想做这个东西的话，最好先了解一下做什么版本，2020年前的版本基本没什么变化，市面上的教程也基本都是针对于27（project1）+80（project2）测试点的版本，我使用的是27+80测试点的代码框架，但是后面把测试集更改成了27+96版本的测试集。最后的结果是project1 27pass，project2 93pass。最开始的文档我使用word写的，所以md文档有些地方可能不太清楚，如果没太明白可以再看一下我的pdf（PINTOS_GUIDE）文件。这个东西前后做了有将近两个月，参考了市面上能找到的中文教程（主要是没有找到英文的教程）。如果出现问题，欢迎再issue下面提问。

* 注意：
* 本文档及代码仅供学习参考，严禁进行商业交易！！  
* 如果有学弟学妹看到了这个报告，希望也不要简单的贴代码，至少按着流程走一遍！！

## 第一章 实验项目介绍

### 总体要求

对于学校要求，我需要至少完成hw-shell、scheduling的task1、task2，后面的部分是选做。

### 实验环境配置步骤

Pintos的安装我是按照网上的教程进行的。步骤如下：
- 1、首先进入virtualBox官网https://www.virtualbox.org/，点击Downloads选项。
- 2、然后选择需要的版本进行下载，然后安装即可。
- 3、然后来到https://ubuntu.com/download/desktop进行Ubuntu的下载。
- 4、紧接着搜索18.04，点击相应版本进行下载。
- 5、下载后进行虚拟机安装，这部分可以参考网上Ubuntu安装方法。
- 6、接下来进行pintos安装，首先来到pintos官网，下载网址如下
https://pintos-os.org/cgi-bin/gitweb.cgi?p=pintos-anon;a=tree;h=refs/heads/master;hb=refs/heads/master
- 7、然后点击snapshot，进行下载。
- 8、然后再虚拟机桌面将压缩包解压，之后需要进行路径的修改，确保pintos能在虚拟机上启动。
- 9、我参考了这个帖子：https://blog.csdn.net/geeeeeker/article/details/108104466，首先在命令行中输入sudo apt-get install qemu
- 10、然后进入pintos里的/utils/pintos-gdb，把pintos完整路径给GDBMARCRO。
- 11、把Makefile文件中LAODLIBES改为LDLBIS。
-	12、这个时候在utils文件夹中使用make命令。
-	13、然后编辑/src/threads/Make.vars中把7行的bochs改为qemu。
-	14、在/utils/pintos中把103行bochs改为qemu，257行kernel.bin改成完整路径，621行改为qemu-system-x86_64
-	15、然后在/utils/Pintos.pm中把loader.bin改为它的完整路径。
-	16、在终端输入source ~/.bashrc运行。
	至此，project部分的配置完成。
	对于shell部分，可以在直接在berkeley的github上进行创建地址如下：https://github.com/Berkeley-CS162/vagrant/

### 问题及解决方法

这里简要说一下做project1和project2的时候出现的问题。
在做project1的时候，一开始出现了booting not properly的输出，原因是因为pintos虚拟机没有跑起来，如果遇到了这种情况，很大的几率是本地的环境配置没写对。然后我上了YouTube看了一位主播的教程，然后按照我如上的方式完成了配置。

在做project2的时候，我再次出现了这种情况，解决办法是把userprog文档下的makefile.var修改掉。在我最后合并project1和2的时候，我发现utils目录下面的pintos和Pintos.pm文件里面的loader和kernel部分的路径变量不需要写死，不然会出现一个project跑不了，另一个却能跑的情况。这个地方我有点忘记之前是怎么更改utils的了，反正如果你做到了project2的时候，注意一下这点就行。

## 第二章 简单shell实现

本章时Pintos项目中的一个个人作业，相比于之后的project来讲比较简单，特别是做完project2后在会过来看shell这个作业就觉得时很容易的，这一部分我是按照Pintos手册里面所给的顺序进行完成的，所以每个部分的名字也是Pintos手册的任务名。在这个作业里，我的任务就是利用现有的接口来实现一个shell的程序，主要的任务就是读取命令行，然后做相关的操作。

### 1、启动任务(getting started)

配置好环境之后进入vagrant用户下面的personal文件夹，编译文件然后运行shell文件即可。

### 2、cd 与 pwd的实现(Add support for cd and pwd)

要求：pwd打印当前的工作路径，cd用来改变工作路径

实现流程：

首先我查看了shell文件下面的函数，找到了已经实现了的exit和help的shell命令，于是按照格式申明了cd和pwd的函数原型

```
int cmd_exit(struct tokens *tokens);
int cmd_help(struct tokens *tokens);
int cmd_pwd(struct tokens *tokens);
int cmd_cd(struct tokens *tokens);
```

进一步的，我对help函数要打印的table内容进行了修改，如下所示(已修改)：

```
fun_desc_t cmd_table[] = {
  {cmd_help, "?", "show this help menu"},
  {cmd_exit, "exit", "exit the command shell"},
  {cmd_pwd, "pwd", "print the current working directory"},
  {cmd_cd, "cd", "change the current working directory"},
  {cmd_wait, "wait", "wait for all background processes to finish"}
};
```

之后进入shell，执行help命令。可以看到help指令被正确地打印了出来。接下来该实现cd和pwd函数了。

#### cd的实现

观察main函数发现，shell会不停地扫描行输入，然后得到token，如果该token在命令表里面，则进入该函数。

在tokenizer.h文件里面我看到了需要的函数tokens\_get\_token，我可以通过这个函数拿到第一个token，然后来记录路径目标。实现请参考源代码cmd_cd函数部分。

思路：这里的实现中有这样几个原则，只识别一个token，当出现多个token（词组）的时候，就认定语法错误，当无输入或者输入为'~'，则进入home目录下的主文件，当只有一个token，则调用chdir函数进行path的变换，如果失败则返回-1并给出提示。实现如下：

```
int cmd_cd(struct tokens *tokens) {
  char *targ_dir = tokens_get_token(tokens, 1);
  if (targ_dir == NULL || strcmp(targ_dir, "~") == 0) {
    targ_dir = getenv("HOME");
    if (targ_dir == NULL) {
      printf("Error changing directory\n");
      return -1;
    }
  }
  int success = chdir(targ_dir);
  if (success == -1) {
    printf("Error changing directory\n");
  }
  return success;
}
```

#### pwd的实现

pwd的实现很简单，只需要使用getcwd进行一下路径的获取，然后输出结果就好了，但是有一点容易被忘记，就是getcwd返回的是一个malloc得来的值，所以需要手动释放内存。实现如下：
```
int cmd_pwd(unused struct tokens *tokens) {
  char cur_dir[4096];
  if (getcwd(cur_dir, 4096) == NULL) {
    printf("Error printing current directory\n");
  }
  printf("%s\n", cur_dir);
  return 1;
}
```

### 3、程序执行(Program execution)

题目要求不能使用execvp，并且希望最终运行格式为：/usr/bin/wc shell.c

作业中提示需要用启用一个子进程，父进程必须等待子进程直至其结束。

在作业文档里告诉了我们wc是一个程序，并且由于现阶段shell并没有完善，所以必须使用完整路径，进一步查阅资料，我得知wc是文档统计程序，会打印文档的相关信息。

execv和execvp的作用均是执行文件，但是execvp的参数是一根文件名和一个arg的数组，而exec是一个路径和一个arg数组，所以execvp可以方便的找到路径。

通过上面的分析，我明白了自己需要做的事情，把输入的token分解成多个小的token，然后第一个是路径，后面的则是arg数组，代表待执行文件。

这里需要注意的事情是，execv中，arg参数的填写应当包含路径在内的所有token。
我用函数parse\_arg来进行token的拆解。

接下来就是运行程序的函数了，主要思路就是拆分token然后调用fork，如果fork调用成功，那么调用execv来执行程序。


### 4、路径解析(Path resolution)

文档中接下来引出了一个问题，如果我每次都要写完整路径的话，这将是非常繁琐的，所以我需要想个办法帮我获取完整的路径来执行文件。这个题目依旧是不允许使用execvp的，因为使用了，这道题目就完全没有了意义。另外，为了判断用户输入的是否已经是完整路径，我使用了access函数，这个函数将会在用户输入完整路径时返回-1。

通过查阅资料，我发现getenv和strtok正好可以帮助到我完成这道题目。

我用这个函数替换了在run\_program里面出现的execv函数，这下我可以直接写文件名来进行任务的执行了。

### 5、输入输出的重定向(Input/Output Redirection)


输入输出型号的格式在Pintos手册中已经给出，所以我只需要检查是否有'<'或者'>'，然后定义是输入还是输出流。Pintos手册上不要求我们对'>>'或者'<<'进行实现，这就降低了一些难度，那么现在有做的事情就是读入一个"[process] > [file]"格式的的命令行，然后进行执行就好了。
重定向可以单独做一个函数，如下所示：
```
int redirect(int old_fd, int new_fd) {
  /*equal to IF FD IS VALID->COPY->CLOSE*/
  if (old_fd == -1 || dup2(old_fd, new_fd) == -1 || close(old_fd) == -1) {
    return -1;
  }
  return 1;
}
```
然后再在主函数里面进行判断，如下所示(判断部分代码)：
```
if (redirect_stdin) {
        int fd = open(token, O_RDONLY);
        if (redirect(fd, STDIN_FILENO) == -1) {
          printf("Error with input %s\n", token);
          exit(-1);
        }
        redirect_stdin = 0;
      } else if (redirect_stdout) {
        int fd = creat(token, S_IRUSR|S_IWUSR|S_IRGRP|S_IWGRP|S_IROTH);
        if (redirect(fd, STDOUT_FILENO) == -1) {
          printf("Error with input %s\n", token);
          exit(-1);
        }
        redirect_stdout = 0;
      } else if (strcmp(token, "<") == 0) {
        redirect_stdin = 1;
      } else if (strcmp(token, ">") == 0) {
        redirect_stdout = 1;
      }
```

### 6 、信号控制(Signal Handling and Terminal Control)

大多数shell都可以让我们停止或者暂停进程，这是因为有特殊的token流，比如说Ctrl-C或者是Ctrl-Z，通过发送信号给shell的子进程来完成这些操作。接下来是一些基本的概念。

#### 6.1、进程组(Process groups)

每个进程都有自己的PID，每个进程也有自己的进程组ID，也叫做PGID。一般来讲，父进程一样的进程之间有相同的PGID，我们可以用getpgid、setpgid、getpgrp等函数来对id进行操作。
注意，当你每次吧一个子进程移动到一个进程组的时候，PGID应该等于PID。

#### 6.2、前台终止(Foreground terminal)

每个终端都有一个前台进程的组ID，当你用CRTL-C，你的终端会给每一个前台的进程组中的进程发送信号。这里，你可以使用终端命令tcsetpgrp，对于标准输入，参数fd应该为0.

#### 6.3、信号概要(Overview of signals)

这里有一些定义了的信号(这里的信号的具体介绍不进行赘述)：SIGINT、SIGQUIT、SIGKILL、SIGTERM、SIGTSTP、SIGCONT、SIGTTIN。

#### 6.4、任务(Task)

我的任务是确保每个程序在开始的时候都有它自己的进程组，当我开始一个进程的时候，进程组应该在前台，结束信号量只能影响前台的程序，而不是后台的shell。
实现如下：
```
    for (int i = 0; i < sizeof(ignore_signals) / sizeof(int); i++) {
      signal(ignore_signals[i], SIG_DFL);
    }
```

### 7、后台运行(Background processing)

当命令行最后有'&'符号的时候，我需要将这个程序放在后台操作。我需要加一个内置的命令wait，函数会等待直到所有的后台工作全部完成。
这个实现是比较简单的，就直接在每句话的最后加一个检测'&'的判断就行，判断如下：
```
int run_bg = length > 1 && strcmp(tokens_get_token(tokens, length - 1), "&") == 0;
```
可以看到，有两个约束，一个是命令长度要大于1，然后就是最后的那个token是'&'。
```
    if (!run_bg) {
      // move to foreground if input doesn't end with "&"
      tcsetpgrp(shell_terminal, getpgrp());
    }
```
如果不是后台，就把他加入前台进程组。

### 8、问题及解决办法

因为Shell部分并不需要进行特别的环境变量的配置，所以做这个作业的时候并没有特别多的问题，唯一遇到的问题就是，刚做这个作业的时候，对Linux还不是很熟，导致ls、pwd这样的命令对我来说很陌生。解决办法就是读Pintos手册，但是相较于后面project的描述，这一部分的手册的内容就显得很少了，不过大部分的只是都能在Linux的书本上找到。 

## 第三章 线程管理

### Pintos线程管理框架介绍

pintos已经为我提供了基本的线程创建、操作的接口，在其中包括了内存分配、线程的基本定义、时间片的设置、锁和信号量的设置等。根据pintos手册的提示，我得知在project1部分，我需要进行修改的文件是threads文件夹和device文件夹中的代码。在pintos手册的FAQ部分中我得知，我需要关注的主要是thread.c、thread.h、sysch.c、timer.c、fixed-point.h（当前源码中并没有）部分的代码。

在文档中，已经比较清晰地介绍了当前pintos代码的问题，首先是代码功能是不完整的，具体分为下面三个，这也是之后我需要解决的问题：

1）当前的计时器代码timer.c中关于中断的代码（sleep相关的函数）虽然已经给出，但是它们是用忙等方式实现的，这十分低效，所以我需要对其进行修改，通过时间片来进一步管理线程。

2）当前的线程调度是有问题的，在pintos的手册里提出了优先级捐赠这个概念，这点在pintos手册里面有详细地讲解，我会在任务2部分仔细地解释。

3）当我们考虑I/O操作的时候，就出现了一个问题，有一些线程的任务会设计很多的I/O操作，而有些不会，这样的区别会直接体现在CPU的占用上，为了平衡这样的区别，pintos的第三个任务让我完成一个Advanced Scheduler这样的功能，旨在减小系统的平均响应时间。

最后，pintos中的thread是以栈的形式进行存储的，从4KB的栈顶往0KB方向生长，而从0KB开始的部分存储的是内核kernel的相关文件，这也意味着，线程的存储数量是有限的，并且每一个线程的大小也不能特别大，所以像直接申请大数组的操作是绝对不能允许的，这会导致内存溢出从而使内核异常。

### 介绍主要函数的功能及实现流程

关于三个task，他们的逻辑关系是从忙等到支持抢占，然后到支持优先级赠予的抢占调度，然后到不支持优先级赠予但是支持多级反馈调度的调度。
	其中抢占实现就是每个规定时间片后检查一下所有进程的优先级，优先级大的放队列前面，变成一个优先队列，然后永远跑第一个进程元素。优先级赠与则是上锁的时候把共享一个锁的一系列进程优先级整体提升，然后重新调度，多级反馈优先级的实现则和上课讲的稍微有一点不一样，上课的时候老师说是建立多个优先级队列，然后当一个进程没有做完就惩罚移动至优先级更低的队列里去，不过Pintos中是直接用公式来对优先级进行刻画，使得优先级慢慢变低。
	在这个project里，我自己创建的主要的函数有：
-	Void blocked_thread_check 
-	Bool thread_cmp_priority 
-	Void thread_hold_the_lock
-	Void thread_remove_lock
-	Void thread_donate_priority
-	Void thread_donate_update
-	Void thread_mlfqs_increase_recent_cpu_by_one
-	Void thread_mlfqs_update_priority
-	Void thread_mlfqs_update_load_avg_and_recent_cpu


### 任务1：Alarm Clock

#### 任务描述

在device/timer.c中，有一个叫作timer\_sleep()的函数，它的功能是用来对线程中一些需要计时的操作进行处理，但是目前的版本使很低效的，因为它在不停地调用thread\_yeild()函数，直到时间片使用完才会退出（忙等）。我现在的任务是更改这个函数，提高工作效率。

#### 实验过程

从需要修改的代码出发，我先查看了timer\_sleep()，代码如下：

```
void
timer_sleep (int64_t ticks) 
{
  int64_t start = timer_ticks ();

  ASSERT (intr_get_level () == INTR_ON);
  while (timer_elapsed (start) < ticks) 
    thread_yield ();
}
```

可以看到，第4行获取了时间片的数量，94行是断言，确保该线程可以中断，后面是忙等操作。

对于现在的忙等，根据之前学过的知识，我可以使用堵塞来避免这样的操作。那么我的方案是：当一个线程需要进行睡眠的时候，我可以把它放入堵塞队列，并且等待它所睡眠的时间，然后再唤醒它，把他加入就绪队列。

于是我首先需要对thread的结构体进行修改。我在thread结构体里加入了ticks\_blocked，用来记录线程被阻塞的时间片的个数。

之后，我在thread的创建函数中初始化了这个值，初值设置为0，初始创建的函数是create\_thread（），于是在这个函数里需要进行初始化。

好的，这下我已经能够记录线程的阻塞的时间片信息了，然后是考虑如何把这个信息用在具体的代码控制里。可以知道，当ticks\_blocked\&gt;0的时候，说明线程还在被堵塞，当ticks\_blocked\&lt;=0的时候，说明线程需要被唤醒，所以这个时候需要修改它的状态，于是我在thread.c中加入了一个新函数blocked\_thread\_check,定义如下所示：

```
/* Check the blocked thread */
void
blocked_thread_check (struct thread *t, void *aux UNUSED)
{
  if (t->status == THREAD_BLOCKED && t->ticks_blocked > 0)
  {
      t->ticks_blocked--;
      if (t->ticks_blocked == 0)
      {
          thread_unblock(t);
      }
  }
}
```

这个函数的意思就是当时间片大于零的时候就把时间片减一，如果时间片等于零，那么就唤醒这个线程。

现在的问题是，线程在哪个地方更新它的时间片数目呢？我仔细研究了一下代码，发现了这样一件事情，timer是每秒钟都会进行100次中断检查的，所以只需要在它采取的检查函数里进行时间片更新就可以了。我是通过以下这个函数发现这个事情的：

```
/* Sets up the timer to interrupt TIMER_FREQ times per second,
   and registers the corresponding interrupt. */
void
timer_init (void) 
{
  pit_configure_channel (0, 2, TIMER_FREQ);
  intr_register_ext (0x20, timer_interrupt, "8254 Timer");
}
```

以上的函数时在pintos开始工作的时候就调用的函数，之后它会使用intr\_register\_ext中的handler来进行中断检查。

根据上面的函数，我得知需要在timer\_interrupt函数中来对中断的时间片进行更新。

但是紧接着又有一个问题，我该如何遍历所有的线程呢？通过查阅pintos手册，我发现thread\_foreach函数是可以用来遍历并操作所有的线程的，另外，所有的线程被统一的管理在all\_list这个链表指针上。

于是我在timer\_interrupt函数中进行了修改，进行时间片的更新，更新后的函数如下：

```
/* Timer interrupt handler. */
static void
timer_interrupt (struct intr_frame *args UNUSED)
{
  ticks++;
  enum intr_level old_level = intr_disable ();
  if (thread_mlfqs)
  {
    thread_mlfqs_increase_recent_cpu_by_one ();
    if (ticks % TIMER_FREQ == 0)
      thread_mlfqs_update_load_avg_and_recent_cpu ();
    else if (ticks % 4 == 0)
      thread_mlfqs_update_priority (thread_current ());
  }
  thread_foreach (blocked_thread_check, NULL);
  intr_set_level (old_level);
  thread_tick ();
}
```

在timer\_interrupt里面，我直接使用了thread\_foreach来对所有的thread进行了遍历，并且用我之前写的blocked\_thread\_check函数进行更新。这样一来，线程睡眠的操作就算是完成了。然后我进行了打分测试，alarm的测试点基本通过，除了alarm-priority ，我开始有点疑惑，然后我看了测试用例，发现这牵扯到了优先级，所以严格意义上来讲，它并不属于alarm部分的内容，所以这里先不讲解这个测试点的实现了（但本任务的实验结果部分已经将这个测试点完成）。

不过我仔细研究了以下为什么当前的代码不能通过这个检测点，发现了原因，当前的代码中并没有对线程的优先级进行排序处理，反而是一股脑的往后面扔，目前里的代码，所有对线程表和线程的操作都是list\_push\_back，这自然没有什么优先级可言。

所以我接下来要干的事情就是要向一个办法修改这个函数或者是用别的函数来对线程的链表进行维护。这也算是我下一个任务的突破口。

至此，任务一结束。

#### 问题与解决方法

在完成第一部分的时候，我很长时间都卡在&quot;如何更新堵塞时间片&quot;这个问题上了，解决的办法就是查看pintos的手册以及指南（里面指出很多重要操作会出现在timer\_interrupt函数里面）。这里有个当时一直有误解的地方，就是老师给的指南里面并没有特别多的信息，对这个project描述最多的应该是CS162的那个手册，但是封面的名字应该叫《CS162 Project 2：Scheduling》，Project2误导了我，让我以为里面并没有我需要的东西。可实际上里面把所有的函数和定义讲得都很明白。我想这导致我花了很长时间来找资料读懂代码。

### 任务2：Priority Scheduling（优先级调度）

#### 任务描述

在Pintos里，每个线程都会有一个优先级（priority value），来对线程进行调度。关于优先级的取值，可以从Pintos的源码中发现，优先级的值域是零到六十三，其中有三个宏定义PRI\_MIN、PRI\_DEFAULT、PRI\_MAX，分别代表0、31、63，优先级从低到高。

根据Pintos手册，我需要修改三个Pintos中的原始的同步操作（lock、semaphore、condition variable）来使得现在的程序能够进行优先级的调度。

在手册中，提出了锁的优先级捐赠的思想（priority donation），其解释如下：

试想有这样一个场景，现在有三个进程A、B、C，优先级是A\&gt;B\&gt;C，并且A和C公用了一个锁——但C已经获得了这个锁——这意味着，A必须等待C做完之后才能做A本身，但是B的优先级大于C，所以A必须等待B做完，再做完C之后最后再来做A。这显然是不符合优先调度的初衷的。所以这里提出了一种解决方案，就是当遇到这种情况的时候，A会让与自己公用锁的优先级比自己第的线程的优先级上升，在这里C就会被提升优先级，最后的结果就会变成先执行C，然后执行A，最后执行B。这也就是我们所说的优先级捐赠。

在手册里描述了要求完成的优先级捐赠的三个特性，第一个是，捐赠者可以是多个，第二个是，当锁被释放的时候，被捐赠的对象优先级会被还原，第三个是，捐赠是支持递归的。

#### 实验过程

在完成这个任务之前，首先我需要先把钱一个任务的最后一个监测点解决掉——alarm-priority。

上个任务最后，我已经发现产生问题的原因了——线程调度的时候并不是以优先队列来进行调度的，而是以一个常规的队列进行调度的，即不管优先级如何，都是先进先出。

这个原因是由list\_push\_back函数导致的。所以我需要对这一部分进行修改。

这个地方我有查看了Pintos手册，手册在overview部分提到了一些我可能需要使用的文件，其中有lib部分，手册中讲到，在lib中可能有我需要用到的函数。我简单的看了一下，发现了一个比较有用的排序函数list\_insert\_ordered。这个函数需要我给一个链表的表头和一个加入的元素，还有一个排序的函数指针，之后这个函数就会根据我给出的规则进行插入。

接下来我还需要写一个判断大小的函数，来告诉这个函数排序的规则。实现如下：
```
/* Priority compare function. */
bool
thread_cmp_priority (const struct list_elem *a, const struct list_elem *b, void *aux UNUSED)
{
  return list_entry(a, struct thread, elem)->priority > list_entry(b, struct thread, elem)->priority;
}
```

上面的函数会返回线程的优先级的比较结果，如果是大于则会使true，反之为false。接下来就是完成优先队列了。

当前的代码中设计优先级插入的只有线程初始化、唤醒两个操作（唤醒有两个相关函数）有涉及，对应的函数是ini\_thread函数、thread\_unblock函数、thread\_yield函数，于是我把其中的list\_push\_back函数全部换成了list\_insert\_ordered函数。这下线程的排序就是以优先级来进行降序排序的了。

之后我单独使用Pintos测试了alarm-priority测试点，完成了测试。

接下来应该开始做donate部分了，但是有一件事情我还是比较在意，preempt在英文中的意思是抢占的意思，这貌似是我之前实现过的东西，但是我在线上平台测试的时候并没有通过，然后我又看了一下我的代码，我发现我的确有优先队列，但是测试用例中使用了create\_thread函数，这个函数是我没有修改过的函数，当前这个函数只是单纯地设置优先级，但是不会考虑设置之后的调度问题。而preempt测试中，在main线程中创建了thread2，但是thread2的优先级较高，所以会把main线程阻塞，所以可以知道在thread\_set\_priority函数调用的时候是需要进行线程重新调度的，所以这个时候需要调用thread\_yeild，于是我在thread\_set\_priority函数中添加了thread\_yeild函数，并且由于如果刚进来的线程优先级大于当前线程是可以阻塞当前线程的，所以我在create\_thread函数里面也添加了thread\_yeild函数，这样一来，一个函数在被创建的时候会先调用unblock函数，将他加入到就绪队列里，并且按照优先级排序，然后使用thread\_yeild进行一次重新的线程调度。

在进行了这一部分的修改后进行测试，priority\_change、priority\_preempt、priority\_fifo三个测试通过了。

接下来就正式进行捐赠优先级的编写了。

根据参考的网上资料，我先对几个基本的测试用例进行了分析。这样能够帮助我更好地明白该如何修改代码框架。。

首先来看priority-donate-one。

```
/* The main thread acquires a lock.  Then it creates two
   higher-priority threads that block acquiring the lock, causing
   them to donate their priorities to the main thread.  When the
   main thread releases the lock, the other threads should
   acquire it in priority order.

   Based on a test originally submitted for Stanford's CS 140 in
   winter 1999 by Matt Franklin <startled@leland.stanford.edu>,
   Greg Hutchins <gmh@leland.stanford.edu>, Yu Ping Hu
   <yph@cs.stanford.edu>.  Modified by arens. */

#include <stdio.h>
#include "tests/threads/tests.h"
#include "threads/init.h"
#include "threads/synch.h"
#include "threads/thread.h"

static thread_func acquire1_thread_func;
static thread_func acquire2_thread_func;

void
test_priority_donate_one (void) 
{
  struct lock lock;

  /* This test does not work with the MLFQS. */
  ASSERT (!thread_mlfqs);

  /* Make sure our priority is the default. */
  ASSERT (thread_get_priority () == PRI_DEFAULT);

  lock_init (&lock);
  lock_acquire (&lock);
  thread_create ("acquire1", PRI_DEFAULT + 1, acquire1_thread_func, &lock);
  msg ("This thread should have priority %d.  Actual priority: %d.",
       PRI_DEFAULT + 1, thread_get_priority ());
  thread_create ("acquire2", PRI_DEFAULT + 2, acquire2_thread_func, &lock);
  msg ("This thread should have priority %d.  Actual priority: %d.",
       PRI_DEFAULT + 2, thread_get_priority ());
  lock_release (&lock);
  msg ("acquire2, acquire1 must already have finished, in that order.");
  msg ("This should be the last line before finishing this test.");
}

static void
acquire1_thread_func (void *lock_) 
{
  struct lock *lock = lock_;

  lock_acquire (lock);
  msg ("acquire1: got the lock");
  lock_release (lock);
  msg ("acquire1: done");
}

static void
acquire2_thread_func (void *lock_) 
{
  struct lock *lock = lock_;

  lock_acquire (lock);
  msg ("acquire2: got the lock");
  lock_release (lock);
  msg ("acquire2: done");
}

```

在第一个测试用例中，主程序的线程先获得了一个锁，紧接着出现了两个比主线程优先级分别高1和2的两个线程，并且一次对锁进行了申请。可以知道，这里测试点是希望测试优先级捐赠的实现。主线程得到锁以后，创建了优先级高1的线程1，当线程1申请锁的时候被阻塞，这时，因为线程1的优先级高于主线程，所以会向主线程进行优先级捐赠，于是主线程的优先级应当上升1，然后主线程创建了线程2，在线程2里，线程申请锁的时候又被阻塞，同样也对主线程优先级进行了捐赠，这时主线程的优先级就和线程2一样了，并且大于线程1的优先级。当主线程释放锁后，线程2和线程1也依次进行锁的获得和释放。

接下来再来分析第二个测试用例priority-donate-multiple：

```
/* The main thread acquires locks A and B, then it creates two
   higher-priority threads.  Each of these threads blocks
   acquiring one of the locks and thus donate their priority to
   the main thread.  The main thread releases the locks in turn
   and relinquishes its donated priorities.
   
   Based on a test originally submitted for Stanford's CS 140 in
   winter 1999 by Matt Franklin <startled@leland.stanford.edu>,
   Greg Hutchins <gmh@leland.stanford.edu>, Yu Ping Hu
   <yph@cs.stanford.edu>.  Modified by arens. */

#include <stdio.h>
#include "tests/threads/tests.h"
#include "threads/init.h"
#include "threads/synch.h"
#include "threads/thread.h"

static thread_func a_thread_func;
static thread_func b_thread_func;

void
test_priority_donate_multiple (void) 
{
  struct lock a, b;

  /* This test does not work with the MLFQS. */
  ASSERT (!thread_mlfqs);

  /* Make sure our priority is the default. */
  ASSERT (thread_get_priority () == PRI_DEFAULT);

  lock_init (&a);
  lock_init (&b);

  lock_acquire (&a);
  lock_acquire (&b);

  thread_create ("a", PRI_DEFAULT + 1, a_thread_func, &a);
  msg ("Main thread should have priority %d.  Actual priority: %d.",
       PRI_DEFAULT + 1, thread_get_priority ());

  thread_create ("b", PRI_DEFAULT + 2, b_thread_func, &b);
  msg ("Main thread should have priority %d.  Actual priority: %d.",
       PRI_DEFAULT + 2, thread_get_priority ());

  lock_release (&b);
  msg ("Thread b should have just finished.");
  msg ("Main thread should have priority %d.  Actual priority: %d.",
       PRI_DEFAULT + 1, thread_get_priority ());

  lock_release (&a);
  msg ("Thread a should have just finished.");
  msg ("Main thread should have priority %d.  Actual priority: %d.",
       PRI_DEFAULT, thread_get_priority ());
}

static void
a_thread_func (void *lock_) 
{
  struct lock *lock = lock_;

  lock_acquire (lock);
  msg ("Thread a acquired lock a.");
  lock_release (lock);
  msg ("Thread a finished.");
}

static void
b_thread_func (void *lock_) 
{
  struct lock *lock = lock_;

  lock_acquire (lock);
  msg ("Thread b acquired lock b.");
  lock_release (lock);
  msg ("Thread b finished.");
}

```

这个测试用例和第一个测试用例差不多，区别就在这里专门多了一个测试释放锁时主线程的优先级状况的输出。这是为了测试是否有考虑释放锁时优先级捐赠的回收情况，当优先级捐赠的捐赠者线程释放锁的时候，应当把它捐赠过优先级的线程进行优先级的还原。

接下来看测试用例priority-donate-lower：

```
/* The main thread acquires a lock.  Then it creates a
   higher-priority thread that blocks acquiring the lock, causing
   it to donate their priorities to the main thread.  The main
   thread attempts to lower its priority, which should not take
   effect until the donation is released. */

#include <stdio.h>
#include "tests/threads/tests.h"
#include "threads/init.h"
#include "threads/synch.h"
#include "threads/thread.h"

static thread_func acquire_thread_func;

void
test_priority_donate_lower (void) 
{
  struct lock lock;

  /* This test does not work with the MLFQS. */
  ASSERT (!thread_mlfqs);

  /* Make sure our priority is the default. */
  ASSERT (thread_get_priority () == PRI_DEFAULT);

  lock_init (&lock);
  lock_acquire (&lock);
  thread_create ("acquire", PRI_DEFAULT + 10, acquire_thread_func, &lock);
  msg ("Main thread should have priority %d.  Actual priority: %d.",
       PRI_DEFAULT + 10, thread_get_priority ());

  msg ("Lowering base priority...");
  thread_set_priority (PRI_DEFAULT - 10);
  msg ("Main thread should have priority %d.  Actual priority: %d.",
       PRI_DEFAULT + 10, thread_get_priority ());
  lock_release (&lock);
  msg ("acquire must already have finished.");
  msg ("Main thread should have priority %d.  Actual priority: %d.",
       PRI_DEFAULT - 10, thread_get_priority ());
}

static void
acquire_thread_func (void *lock_) 
{
  struct lock *lock = lock_;

  lock_acquire (lock);
  msg ("acquire: got the lock");
  lock_release (lock);
  msg ("acquire: done");
}

```

这个测试用例里，主线程创建了一个比自己大十优先级的线程1，线程1获取锁的时候被阻塞，赠予主线程优先级，然后对自己进行了优先级降级，但是由于自己是被捐赠优先级的线程，所以降级此时是无效的。当主线程释放锁的时候，线程1得到并释放锁，之后主线程的优先级编程了降级后的优先级。这个测试用例告诉我，一个被捐赠优先级的线程是无法被降级的，直到捐赠被回收。

接下来再看优先级捐赠测试用例里面最复杂的一个测试用例，priority-donate-chain：

```
/* The main thread set its priority to PRI_MIN and creates 7 threads 
   (thread 1..7) with priorities PRI_MIN + 3, 6, 9, 12, ...
   The main thread initializes 8 locks: lock 0..7 and acquires lock 0.

   When thread[i] starts, it first acquires lock[i] (unless i == 7.)
   Subsequently, thread[i] attempts to acquire lock[i-1], which is held by
   thread[i-1], except for lock[0], which is held by the main thread.
   Because the lock is held, thread[i] donates its priority to thread[i-1],
   which donates to thread[i-2], and so on until the main thread
   receives the donation.

   After threads[1..7] have been created and are blocked on locks[0..7],
   the main thread releases lock[0], unblocking thread[1], and being
   preempted by it.
   Thread[1] then completes acquiring lock[0], then releases lock[0],
   then releases lock[1], unblocking thread[2], etc.
   Thread[7] finally acquires & releases lock[7] and exits, allowing 
   thread[6], then thread[5] etc. to run and exit until finally the 
   main thread exits.

   In addition, interloper threads are created at priority levels
   p = PRI_MIN + 2, 5, 8, 11, ... which should not be run until the 
   corresponding thread with priority p + 1 has finished.
  
   Written by Godmar Back <gback@cs.vt.edu> */ 

#include <stdio.h>
#include "tests/threads/tests.h"
#include "threads/init.h"
#include "threads/synch.h"
#include "threads/thread.h"

#define NESTING_DEPTH 8

struct lock_pair
  {
    struct lock *second;
    struct lock *first;
  };

static thread_func donor_thread_func;
static thread_func interloper_thread_func;

void
test_priority_donate_chain (void) 
{
  int i;  
  struct lock locks[NESTING_DEPTH - 1];
  struct lock_pair lock_pairs[NESTING_DEPTH];

  /* This test does not work with the MLFQS. */
  ASSERT (!thread_mlfqs);

  thread_set_priority (PRI_MIN);

  for (i = 0; i < NESTING_DEPTH - 1; i++)
    lock_init (&locks[i]);

  lock_acquire (&locks[0]);
  msg ("%s got lock.", thread_name ());

  for (i = 1; i < NESTING_DEPTH; i++)
    {
      char name[16];
      int thread_priority;

      snprintf (name, sizeof name, "thread %d", i);
      thread_priority = PRI_MIN + i * 3;
      lock_pairs[i].first = i < NESTING_DEPTH - 1 ? locks + i: NULL;
      lock_pairs[i].second = locks + i - 1;

      thread_create (name, thread_priority, donor_thread_func, lock_pairs + i);
      msg ("%s should have priority %d.  Actual priority: %d.",
          thread_name (), thread_priority, thread_get_priority ());

      snprintf (name, sizeof name, "interloper %d", i);
      thread_create (name, thread_priority - 1, interloper_thread_func, NULL);
    }

  lock_release (&locks[0]);
  msg ("%s finishing with priority %d.", thread_name (),
                                         thread_get_priority ());
}

static void
donor_thread_func (void *locks_) 
{
  struct lock_pair *locks = locks_;

  if (locks->first)
    lock_acquire (locks->first);

  lock_acquire (locks->second);
  msg ("%s got lock", thread_name ());

  lock_release (locks->second);
  msg ("%s should have priority %d. Actual priority: %d", 
        thread_name (), (NESTING_DEPTH - 1) * 3,
        thread_get_priority ());

  if (locks->first)
    lock_release (locks->first);

  msg ("%s finishing with priority %d.", thread_name (),
                                         thread_get_priority ());
}

static void
interloper_thread_func (void *arg_ UNUSED)
{
  msg ("%s finished.", thread_name ());
}

// vim: sw=2

```

在这个测试用例里面涉及到了线程的优先级和锁的优先级。主线程先申请到8个锁，然后依次分配给8个线程，然后每个线程会申请前一个序号的线程的锁（主线程除外），可以知道1到7号线程先被阻塞，然后主线程的优先级会因为被捐赠而增高，最后优先级到达21。当主线程释放0号锁的时候，一个第1个线程抢占主线程，当1号线程释放1号锁的时候，2号线程又会进行抢占，这样的情况会持续到七号线程，当锁释放完毕，主线程的优先级又变成了0，于是被循环中的输出线程抢占，在输出线程运行完毕后，主线程最后进行输出，结束程序。

在这个测试用例里面，涉及到了多个锁相互嵌套的问题，这也是需要我解决的部分。

另外，在捐赠模块的测试样例里也存在与信号量相关的测试用例。接下来进行两个结合了信号量的测试用例，首先是priority-donate-sema：

```
/* Low priority thread L acquires a lock, then blocks downing a
   semaphore.  Medium priority thread M then blocks waiting on
   the same semaphore.  Next, high priority thread H attempts to
   acquire the lock, donating its priority to L.

   Next, the main thread ups the semaphore, waking up L.  L
   releases the lock, which wakes up H.  H "up"s the semaphore,
   waking up M.  H terminates, then M, then L, and finally the
   main thread.

   Written by Godmar Back <gback@cs.vt.edu>. */

#include <stdio.h>
#include "tests/threads/tests.h"
#include "threads/init.h"
#include "threads/synch.h"
#include "threads/thread.h"

struct lock_and_sema 
  {
    struct lock lock;
    struct semaphore sema;
  };

static thread_func l_thread_func;
static thread_func m_thread_func;
static thread_func h_thread_func;

void
test_priority_donate_sema (void) 
{
  struct lock_and_sema ls;

  /* This test does not work with the MLFQS. */
  ASSERT (!thread_mlfqs);

  /* Make sure our priority is the default. */
  ASSERT (thread_get_priority () == PRI_DEFAULT);

  lock_init (&ls.lock);
  sema_init (&ls.sema, 0);
  thread_create ("low", PRI_DEFAULT + 1, l_thread_func, &ls);
  thread_create ("med", PRI_DEFAULT + 3, m_thread_func, &ls);
  thread_create ("high", PRI_DEFAULT + 5, h_thread_func, &ls);
  sema_up (&ls.sema);
  msg ("Main thread finished.");
}

static void
l_thread_func (void *ls_) 
{
  struct lock_and_sema *ls = ls_;

  lock_acquire (&ls->lock);
  msg ("Thread L acquired lock.");
  sema_down (&ls->sema);
  msg ("Thread L downed semaphore.");
  lock_release (&ls->lock);
  msg ("Thread L finished.");
}

static void
m_thread_func (void *ls_) 
{
  struct lock_and_sema *ls = ls_;

  sema_down (&ls->sema);
  msg ("Thread M finished.");
}

static void
h_thread_func (void *ls_) 
{
  struct lock_and_sema *ls = ls_;

  lock_acquire (&ls->lock);
  msg ("Thread H acquired lock.");

  sema_up (&ls->sema);
  lock_release (&ls->lock);
  msg ("Thread H finished.");
}

```

这个测试样例结合了同步信号量，一开始在主线程中设置了一个同步信号量和一个锁，然后创建了一个low线程，在low线程中获得锁，然后使用同步信号量，被阻塞，然后回到主线程，主线程创建了med线程，med线程中再次被同步信号量阻塞，回到主线程，创建high线程，high线程中获得锁的时候被阻塞，这是会向low线程捐赠优先级，low线程优先级变成+5。然后回到主线程，主线程使用了同步信号量V操作，low线程在等待队列中被唤醒，紧接着释放了锁，同时优先级回到+1，然后被high线程抢占，high线程得到锁后，做了一个V操作，high结束后，由于med优先级比low高，所以med线程先执行，执行后再low线程，Main线程的优先级是最低的，所以最后执行，程序到此结束。

接下来是最后一个要分析的测试用例，priority-condvar：

```
/* Tests that cond_signal() wakes up the highest-priority thread
   waiting in cond_wait(). */

#include <stdio.h>
#include "tests/threads/tests.h"
#include "threads/init.h"
#include "threads/malloc.h"
#include "threads/synch.h"
#include "threads/thread.h"
#include "devices/timer.h"

static thread_func priority_condvar_thread;
static struct lock lock;
static struct condition condition;

void
test_priority_condvar (void) 
{
  int i;
  
  /* This test does not work with the MLFQS. */
  ASSERT (!thread_mlfqs);

  lock_init (&lock);
  cond_init (&condition);

  thread_set_priority (PRI_MIN);
  for (i = 0; i < 10; i++) 
    {
      int priority = PRI_DEFAULT - (i + 7) % 10 - 1;
      char name[16];
      snprintf (name, sizeof name, "priority %d", priority);
      thread_create (name, priority, priority_condvar_thread, NULL);
    }

  for (i = 0; i < 10; i++) 
    {
      lock_acquire (&lock);
      msg ("Signaling...");
      cond_signal (&condition, &lock);
      lock_release (&lock);
    }
}

static void
priority_condvar_thread (void *aux UNUSED) 
{
  msg ("Thread %s starting.", thread_name ());
  lock_acquire (&lock);
  cond_wait (&condition, &lock);
  msg ("Thread %s woke up.", thread_name ());
  lock_release (&lock);
}

```

在上述用例里面加入了cond\_wait和cond\_signal两个函数，查阅Pintos手册得知，这两个函数是集成了lock和信号量的函数，并且在线程阻塞时，会主动释放当前的lock。在这个程序里面， 主线程创建了多个线程，但是因为用了cond函数，所以并不存在优先级捐赠，然后到第二个循环的时候，主线程释放信号量，使用V操作，唤醒了队列里最前面的线程，继而线程被一一唤醒，顺序则是按优先级排列的。

至此，所有的测试用例被分析完毕。接下来进行实现的描述。

经过上述的测试用例分析，可以知道，对于线程，需要记录自己最开始的优先级，还有与自己相关的锁。对于锁来说，则需要记录最大的优先级，来完成优先级的捐赠，也需要一个链表来记录捐赠过的线程。

于是在thread.h中对thread数据结构添加如下成员变量：

```
int base_priority;
struct list locks;
struct lock *lock_waiting;
```

然后在sysch.h中对lock结构体进行成员变量的添加：

```
struct list_elem elem;
int max_priority;
```

然后开始修改相关的代码。首先来看一下原来锁的获得和释放是怎么写的，这里将原来的代码展示如下：

原始的lock\_acquire函数
```
void
lock_acquire (struct lock *lock)
{
  ASSERT (lock != NULL);
  ASSERT (!intr_context ());
  ASSERT (!lock_held_by_current_thread (lock));

  sema_down (&lock->semaphore);
  lock->holder = current_thread;
}
```
原始的lock\_release函数
```
void
lock_release (struct lock *lock)
{
  ASSERT (lock != NULL);
  ASSERT (!intr_context ());
  ASSERT (!lock_held_by_current_thread (lock));

  lock->holder = NULL;
  sema_up (&lock->semaphore);
}
```
对于锁的获取而言，这里需要添加两个必要的操作。第一个则是优先级的捐赠，第二个则是最大优先级的更新。对于第一个操作而言，如果得知当前的线程优先级是比锁相关的线程中最大的优先级低的，那么就可以直接进行捐赠，这里可以考虑直接在wait\_list中遍历更新。对于第二个操作，经过考虑，应该是放在第一个操作之前的，这其实也很好理解，因为当前的线程也很可能是最大的优先级，这样的话他就需要对所有的相关线程进行优先级捐赠。

对于锁的释放而言，源代码只有对所本身的释放，这其实没有什么问题，但因为我之前已经引入了新的成员变量，所以需要进行队成员变量的维护，其中也包括了捐赠的优先级的收回。

```
void
lock_acquire (struct lock *lock)
{
  struct thread *current_thread = thread_current ();
  struct lock *l;
  enum intr_level old_level;

  ASSERT (lock != NULL);
  ASSERT (!intr_context ());
  ASSERT (!lock_held_by_current_thread (lock));

  if (lock->holder != NULL && !thread_mlfqs)
  {
    current_thread->lock_waiting = lock;
    l = lock;
    while (l && current_thread->priority > l->max_priority)
    {
      l->max_priority = current_thread->priority;
      thread_donate_priority (l->holder);
      l = l->holder->lock_waiting;
    }
  }

  sema_down (&lock->semaphore);

  old_level = intr_disable ();

  current_thread = thread_current ();
  if (!thread_mlfqs)
  {
    current_thread->lock_waiting = NULL;
    lock->max_priority = current_thread->priority;
    thread_hold_the_lock (lock);
  }
  lock->holder = current_thread;

  intr_set_level (old_level);
}
```

上述代码开始先对当前锁的优先级进行更新和对锁涉及的线程进行优先级的更新。然后考虑得到锁后对当前的最大优先级进行更新。

其中已经使用了两个被实现的函数，thread\_donate\_priority、thread\_hold\_lock，现在对这两个函数进行解释。

对于优先级的捐赠，逻辑相对简单，就是把当前最大的优先级赋值给目标的线程。但是这里要注意一件事情，就是赋值之后是否可以抢占，对于我们的测试用例而言是可以的，所以在赋值之后需要添加一个判断分支，对当前的线程进行重新的调度。函数实现如下：

```
/* Donate current priority to thread t. */
void
thread_donate_priority (struct thread *t)
{
  enum intr_level old_level = intr_disable ();
  thread_update_priority (t);

  if (t->status == THREAD_READY)
  {
    list_remove (&t->elem);
     list_insert_ordered (&ready_list, &t->elem, thread_cmp_priority, NULL);
  }
  intr_set_level (old_level);
}
```

这里进行了抢占问题的解决。当前的的线程是就绪队列的线程，则将会对它重新调度。

下面是thread\_hold\_the\_lock函数的实现：

```
/* Let thread hold a lock */
void
thread_hold_the_lock(struct lock *lock)
{
  enum intr_level old_level = intr_disable ();
  list_insert_ordered (&thread_current ()->locks, &lock->elem, lock_cmp_priority, NULL);

  if (lock->max_priority > thread_current ()->priority)
  {
    thread_current ()->priority = lock->max_priority;
    thread_yield ();
  }

  intr_set_level (old_level);
}
```

在632行，对当前线程的锁的顺序进行了排序，因为锁也存在优先级，所以也有先后释放的问题。

对于上面的thread\_update\_priority函数的实现，有以下代码：

```
void
thread_update_priority (struct thread *t)
{
  enum intr_level old_level = intr_disable ();
  int max_priority = t->base_priority;
  int lock_priority;

  if (!list_empty (&t->locks))
  {
    list_sort (&t->locks, lock_cmp_priority, NULL);
    lock_priority = list_entry (list_front (&t->locks), struct lock, elem)->max_priority;
    if (lock_priority > max_priority)
      max_priority = lock_priority;
  }

  t->priority = max_priority;
  intr_set_level (old_level);
}

```

这个函数是为了方便更新函数所写的，可以看到在673行首先将当前线程的优先级变为base\_priority，这也是该线程本来的优先级，然后再676行进行判断，如果该线程有锁，则将它的线程优先级变为它拥有锁中所拥有的最大的优先级，如果没有锁的话那么它的优先级就为base\_priority。

当然上述的locks、lock\_waiting、base\_priority应当在创建线程时定义，所以应当在inti\_thread中进行初始化。

最后，还应当对thread\_set\_priority函数进行修改。还记得刚刚的测试用例吗，其中有一个用例中出现了一种情况，在线程获得锁的时候是不能以捐赠之外的手段对其改变它的优先级的。所以对thread\_set\_priority进行一下更改。

刚改完上述的代码后我进行了测试，但是发现关于信号量的测试点是失败的。我再次检查了源码，发现还需要修改与信号量相关的代码。

当前的cond_signal和sema_up函数看上去好像没有什么不对的地方，但是仔细思考会发现，这里没有了优先级的逻辑，就导致V操作后唤醒的线程不一定是优先级最高的线程。

所以对这两个函数做出以下修改：

```
void
cond_signal (struct condition *cond, struct lock *lock UNUSED)
{
  ASSERT (cond != NULL);
  ASSERT (lock != NULL);
  ASSERT (!intr_context ());
  ASSERT (lock_held_by_current_thread (lock));

  if (!list_empty (&cond->waiters))
  {
    list_sort (&cond->waiters, cond_sema_cmp_priority, NULL);
    sema_up (&list_entry (list_pop_front (&cond->waiters), struct semaphore_elem, elem)->semaphore);
  }
}
```


```
void
sema_up (struct semaphore *sema)
{
  enum intr_level old_level;

  ASSERT (sema != NULL);

  old_level = intr_disable ();
  if (!list_empty (&sema->waiters))
  {
    list_sort (&sema->waiters, thread_cmp_priority, NULL);
    thread_unblock (list_entry (list_pop_front (&sema->waiters), struct thread, elem));
  }

  sema->value++;
  thread_yield ();
  intr_set_level (old_level);
}

```

对上面两个函数的修改起始就是在进行唤醒操作前对信号量、线程的优先级进行排序。具体的排序函数和之前优先级的排序函数基本一致，所以不再赘述。

通过上述的改进后，再次尝试，信号量部分也完成了，至此任务二完成。

#### 实验结果

这时我进行了一次线上的评分，此时是0分，老师解释，错误数量大于5个不计分。

#### 问题与解决方法

在完成这个任务的时候遇到的最头疼的困难就是看测试样例。因为很多需求Pintos手册都没有明确说明，所以想要知道为什么代码这么写和代码应该怎么写完全需要看测试样例。由于任务一已经积累了启动Pintos的经验，所以任务二的调试并没有特别困难。

### 任务3：Advanced Scheduler（多级反馈调度）

#### 任务描述

根据Pintos手册的描述，多级反馈调度的提出是替代优先级捐赠的另一种方案。试想这样一种情况：在操作系统的运行过程中，不同的线程工作的特性是不同的，有一些线程有很多I/O操作，它需要很快的系统响应时间，但它并不需要占用很长时间CPU，而有一些线程的工作可能不需要很多I/O等待，可这就意味着它们需要更长的CPU时间。所以在这种情况下，我们并不能对所有线程的优先级都一视同仁（这里我们不考虑优先级捐赠的方法），所以Pintos手册里给出了一种量化优先级的方法，用nice值和recent\_CPU两个参数来对线程的优先级进行刻画。Pintos已经给出了相应的公式，公式如下：

_priority__ = PRI\_MAX - (__recent\_cpu__ / 4) - (__nice__ \* 2)._

recent\_cpu的意思是线程最近平均使用cpu的时间，他是可以被迭代的。对recent\_cpu的描述如下：

_recent\_cpu__ = (2\*__load\_avg__)/(2\*__load\_avg__ + 1) \* __recent\_cpu __ + __ nice__

那么load\_avg又是什么呢，Pintos手册里进行了解释。load\_avg通常被称作系统平均装载时间（system load average），用来估计线程在过去的时间里平均的运行时间。它是可以被迭代的，迭代公式如下：

_load\_avg__ = (59/60)\*__load\_avg__ + (1/60)\*__ready\_threads__

在这个任务里，我需要做的就是用轮转加nice值估计结合的算法来进行线程调度。这也和上课所讲的差不多，只是这里并没有用额外的队列进行线程优先级的刻画，取而代之的是对优先级的迭代公式，从上述公式可以看出，recent\_cpu迭代次数越多，其值是越来越大的。因此从第一个式子中可以看出它的优先级会越来越低，这也和上课的内容相互契合。

另外，Pintos已经给出了fixed\_point.h的头文件，这是因为Pintos并不支持浮点数运算。

#### 实验过程

虽然是附加的实验，但是思路并不是很难，比任务二简单了不少，因为公式已经给出来了，所以只需要按照公式计算就行了。

可以明确，这里需要对nice和recent\_cpu进行定义，所以我在thread的结构体中加入了这两个成员变量。

```
int nice;
fixed_t recent_cpu;
```

之后我们要做的是先将所有的donate部分的代码进行if判断，如果是mlfqs为真则跳过。这部分的修改是Pintos手册明确要求的。

那么现在要做的事情就是按照手册所说的顺序进行公式的计算就好了，下面是对recent\_cpu加一的函数：

```
/* Increase recent_cpu by 1. */
void
thread_mlfqs_increase_recent_cpu_by_one (void)
{
  ASSERT (thread_mlfqs);
  ASSERT (intr_context ());

  struct thread *current_thread = thread_current ();
  if (current_thread == idle_thread)
    return;
  current_thread->recent_cpu = FP_ADD_MIX (current_thread->recent_cpu, 1);
}
```

我们首先要知道我们应该对所有的线程在给定时间内进行检查，这里我把时间片设置为了4，这个代码

接下来是比较重要的函数，负责两个参数的更新：

```
/* Every per second to refresh load_avg and recent_cpu of all threads. */
void
thread_mlfqs_update_load_avg_and_recent_cpu (void)
{
  ASSERT (thread_mlfqs);
  ASSERT (intr_context ());

  size_t ready_threads = list_size (&ready_list);
  if (thread_current () != idle_thread)
    ready_threads++;
  load_avg = FP_ADD (FP_DIV_MIX (FP_MULT_MIX (load_avg, 59), 60), FP_DIV_MIX (FP_CONST (ready_threads), 60));

  struct thread *t;
  struct list_elem *e = list_begin (&all_list);
  for (; e != list_end (&all_list); e = list_next (e))
  {
    t = list_entry(e, struct thread, allelem);
    if (t != idle_thread)
    {
      t->recent_cpu = FP_ADD_MIX (FP_MULT (FP_DIV (FP_MULT_MIX (load_avg, 2), FP_ADD_MIX (FP_MULT_MIX (load_avg, 2), 1)), t->recent_cpu), t->nice);
      thread_mlfqs_update_priority (t);
    }
  }
}
```

在这个函数对两个参数的计算，其中因为一个分支因为时间片已经耗尽，所以需要进行重新调度。

接下来是最后一个函数，更新线程的优先级。这个函数在刚刚的函数里也调用了，分支为什么要这么写呢，因为Pintos手册里建议更新的时间片长度为TIMER\_FREQ（扩展到100），而原本的多级反馈调度时间片是4个（如果扩展到100），我们的调度很可能会退化成轮转调度。该函数如下所示：

```
/* Update priority. */
void
thread_mlfqs_update_priority (struct thread *t)
{
  if (t == idle_thread)
    return;

  ASSERT (thread_mlfqs);
  ASSERT (t != idle_thread);

  t->priority = FP_INT_PART (FP_SUB_MIX (FP_SUB (FP_CONST (PRI_MAX), FP_DIV_MIX (t->recent_cpu, 4)), 2 * t->nice));
  t->priority = t->priority < PRI_MIN ? PRI_MIN : t->priority;
  t->priority = t->priority > PRI_MAX ? PRI_MAX : t->priority;
}
```

这里需要注意的事情是，得到的优先级很可能是一个不再优先级范围内的数，所以需要进行一下判断（712、713行）。

然后是主体函数，我对timer\_interrupt函数进行了更新，如下所示，时间片定义为4个：

```
/* Timer interrupt handler. */
static void
timer_interrupt (struct intr_frame *args UNUSED)
{
  ticks++;
  enum intr_level old_level = intr_disable ();
  if (thread_mlfqs)
  {
    thread_mlfqs_increase_recent_cpu_by_one ();
    if (ticks % TIMER_FREQ == 0)
      thread_mlfqs_update_load_avg_and_recent_cpu ();
    else if (ticks % 4 == 0)
      thread_mlfqs_update_priority (thread_current ());
  }
  thread_foreach (blocked_thread_check, NULL);
  intr_set_level (old_level);
  thread_tick ();
}
```

当然这还没有结束，因为测试样例中需要我们提供一些查看参数的接口，代码框架中已经存在这些函数了，只需要我们在里面返回一下值就行。

到此为止，任务三完成。

#### 实验结果

线上的测试结果为28分。

#### 问题与解决方法

任务三总体来讲是比较简单的，直接用公式就好了，但是浮点数头文件的理解稍微有点困难，我查了一些网上的资料。

之后就是调试了，一开始我不知道需要使用-mlfqs选项，导致pintos启动的时候直接就错误了，后来查看pintos手册才知道需要用-mlfqs选项启动。

另外，在最后的网上测试的时候也出现了一些问题，就是本地测试没有问题，但是最后会在线上平台超时。

出现这个错误以后我并没有在我的代码里找到逻辑错误，然后我查看了Pintos手册，里面说可能是timer\_interrupt这个函数做的事情太多了，于是我把时间片进行了修改，每20个时间片进行一次更新。但是好像效果还是不行。最后我更改了pit.c的函数，增加了timer的检查次数。

更改后，timer的检查频率会增加，这样就不会出现该调度的时候cpu却什么都没做的情况了。

当然还有另一种解决办法，就是把timer\_interrupt函数里面的时间片由4个变成20或50（更大的时间片），这样自然这个函数里干的事情就会变少。但是这样的做法是不被官方推荐的，这会使得反馈调度的时间周期和规定的不一样，进而导致和结果错误。

最后还有一种办法，就是较少timer_interrupt函数中的开销，做法就是减少链表的遍历次数。原先代码中recent_cpu和update_priority各自遍历了一次链表，所以改进方案为，遍历一次链表，同时更新这两个参数。

## 第四章 用户程序

### Pintos的用户程序（Userprog）框架详细介绍

在现有的Pintos中，有如下文件是需要我去仔细查看的：

threads/thread.h：包含了线程的定义，这里有Pintos中对线程的控制块。另外，在这个Project里，有以下宏定义是需要被使用的。

userprog/process.c：负责装载ELF二进制文件，开启进程，还有切换页表。

userprog/pagedir.c：负责管理页表，我不需要对这部分代码进行修改，但是有一部分的函数我或许是需要使用的。

userprog/syscall.c：这是系统调用的处理部分，在一开始，它只支持exit这样一个系统调用。

lib/user/syscall.c：为用户程序提供了库函数来使用系统调用。每一个函数都是用了内联的汇编代码来应对系统的调用指令和使用喜用调用。

lib/syscall-nr.h：这个文件定义了系统调用的枚举变量。

userprog/excption.c：处理异常，现在的处理方式只是简单地打印信息然后关闭进程。在后面的任务中有一部分会需要我来修改page_fault函数。

gdt.c：Global Descriptor Table是一个解释当前正在使用的短的表。供参考了解。

tss.c：Task-State Segment 会在进入中断处理部分时帮助Pintos变换堆栈。供参考了解。

### pintos内存分配的实现流程和堆栈的使用

在Pintos中，虚拟内存被分为两个区域，用户虚拟内存和内核虚拟内存，用户虚拟内存时从0开始一直到PHYS_BASE(0xc0000000[3GB])，而内核虚拟内存从PHYS_BASE到4GB。

对于虚拟内存而言，它是以每个进程为单位来分配的。这其实也和上课讲的一样，线程时资源分配的基本单位。


当内核切换进程的时候，它会改变调度器中的基址寄存器中的值。内核的虚拟内存是全局的，不管此刻使用的权限是内核还是用户，它们的地址映射都是一样的，虚拟内存地址中的PHYS_BASE对应物理地址的0，而虚拟内存的PHYS_BASE + 0x1234对应物理内存的0x1234。

但是尽管如此，用户程序能访问的地址依然只是它自己的虚拟内存地址。如果它企图去访问内核的虚拟内存地址，那么就会发生page fault。而对于内核而言，企图去访问用户的虚拟内存也会出现page fault。

Pintos手册里介绍了一些具体的内存实现的原理。我总结一下：

- 对于80x86的调用操作：
- 1、调用时将函数的参数一个一个的压栈，通常这一步使用汇编代码。压入的顺序应该是从右到左的。（这样从栈里取出来的时候就是从左到右的）。每一次压栈的时候需要让栈指针自减，在C语言里类似的有：*--sp=value
- 2、调用部分把返回地址的指令(也就是调用此函数的函数的地址)压入堆栈，然后跳转到调用的函数里。简单来讲，就是在调用函数之前把当前的地址保护起来，然后跳转到需要调用的函数地址。
- 3、调用函数的时候，就从当前的sp开始往下面找，栈顶为返回地址，所以跳过栈顶就是函数的参数了。
- 4、调用函数的时候，如果函数有返回值，把它保存在eax中。
- 5、需要退出函数的时候，进行出栈操作，使用栈顶保存的返回地址回到特定的地址位置，这里需要使用80x86的ret命令。
- 6、返回后不要忘记把刚刚压栈的参数也弹出。不然堆栈不停增长，最后就会导致内存溢出。

### 任务1：ArgumentPassing

#### 任务描述

在Pintos代码的process.c文件中已经存在一个函数process\_execute。这个函数是用来创建新的用户级进程的。但是目前Pintos中并不支持参数指令，所以我需要实现参数的传递。类似于process\_execute(&quot;ls -ahl&quot;)这样的两个参数的传入，在程序中使用的标记是argc和argv。

另外需要注意的事情是，因为Pintos中的参数传递还没有被实现，而Pintos提供的测试用例都有自己的名字，所以目前执行测试Pintos会直接崩溃。

所以当前的目的是实现进程执行时候的参数传递。

#### 实验过程

其实刚看到这个任务有点没有头绪，然后我开始看Pintos手册。Pintos手册在这一章主要介绍了一些内存相关的东西，但这些并不是我现在需要去看的东西。所以我来到问题本身，看看现在的代码写了什么。

```
/* Starts a new thread running a user program loaded from
   FILENAME.  The new thread may be scheduled (and may even exit)
   before process_execute() returns.  Returns the new process's
   thread id, or TID_ERROR if the thread cannot be created. */
tid_t
process_execute (const char *file_name) 
{
  char *fn_copy0;
  tid_t tid;

    /* Make a copy of FILE_NAME.
       Otherwise strtok_r will modify the const char *file_name. */
    fn_copy0 = palloc_get_page(0);
    if (fn_copy0 == NULL)//分配失败
        return TID_ERROR;
  strlcpy (fn_copy, file_name, PGSIZE);


  /* Create a new thread to execute FILE_NAME. */
  tid = thread_create(cmd, PRI_DEFAULT, start_process, fn_copy1);
  if (tid == TID_ERROR)
    palloc_free_page (fn_copy1); 
  return tid;

}
```

我们看到在一开始，程序申请了空间赋值了file\_name这个字符串，然后紧接着就创建进程了，这个函数我已经比较熟悉了，上一个project的任务就是围绕thread\_create展开的，带式最后一个参数当时我并没有去管，然后我又回去看thread\_create里面拿它干了什么。发现这个东西最后回赋值给kernel\_thread\_frame结构体里的aux。

这个结构体又三个变量，分别是返回值、函数指针和可选项。根据stack注释可知道这些是压入堆栈使用了。

按照Pintos手册给的例子（我们只实现一个参数的情况），当我们在Pintos中输入&quot;ls -ahl&quot;后，读入&quot;ls&quot;和&quot;-ahl&quot;两个命令，分别作为argc和argv。

这样就发现问题了，在当前的excute\_process函数里面，我们创建线程的时候，对于可选项部分我们直接传了file\_name,这显然是不对的。这会导致上面的例子中，argc和argv都是&quot;ls -ahl&quot;——这样两个参数都错了。

所以我们现在的目标明确了。把传入的参数拆分成两半，然后再给thread\_create函数。逻辑不难，所以给出修改的代码：

```
/* Starts a new thread running a user program loaded from
   FILENAME.  The new thread may be scheduled (and may even exit)
   before process_execute() returns.  Returns the new process's
   thread id, or TID_ERROR if the thread cannot be created. */
tid_t
process_execute (const char *file_name) 
{
  char *fn_copy0, *fn_copy1;
  tid_t tid;


    /* Make a copy of FILE_NAME.
       Otherwise strtok_r will modify the const char *file_name. */
    fn_copy0 = palloc_get_page(0);//palloc_get_page(0)动态分配了一个内存页
    if (fn_copy0 == NULL)//分配失败
        return TID_ERROR;

  /* Make a copy of FILE_NAME.
     Otherwise there's a race between the caller and load(). */
  fn_copy1 = palloc_get_page (0);
  if (fn_copy1 == NULL)
  {
    palloc_free_page(fn_copy0);
    return TID_ERROR;
  }
  //把file_name 复制2份，PGSIZE为页大小
  strlcpy (fn_copy0, file_name, PGSIZE);
  strlcpy (fn_copy1, file_name, PGSIZE);


  /* Create a new thread to execute FILE_NAME. */
  char *save_ptr;
  char *cmd = strtok_r(fn_copy0, " ", &save_ptr);
  
  tid = thread_create(cmd, PRI_DEFAULT, start_process, fn_copy1);
  palloc_free_page(fn_copy0);
  if (tid == TID_ERROR)
  {
    palloc_free_page (fn_copy1); 
    return tid;
  }
    
    /* Sema down the parent process, waiting for child */
  sema_down(&thread_current()->sema);
  if (!thread_current()->success) return TID_ERROR;//can't create new process thread,return error

  return tid;
}
```

在实现这个函数的时候我偷了懒，使用了Pintos中已经提供的函数strtok\_r来进行arguments的分割。这个函数是在string.h中被申明的。函数如下：


```
char *
strtok_r (char *s, const char *delimiters, char **save_ptr) 
{
  char *token;
  
  ASSERT (delimiters != NULL);
  ASSERT (save_ptr != NULL);

  /* If S is nonnull, start from it.
     If S is null, start from saved position. */
  if (s == NULL)
    s = *save_ptr;
  ASSERT (s != NULL);

  /* Skip any DELIMITERS at our current position. */
  while (strchr (delimiters, *s) != NULL) 
    {
      /* strchr() will always return nonnull if we're searching
         for a null byte, because every string contains a null
         byte (at the end). */
      if (*s == '\0')
        {
          *save_ptr = s;
          return NULL;
        }

      s++;
    }

  /* Skip any non-DELIMITERS up to the end of the string. */
  token = s;
  while (strchr (delimiters, *s) == NULL)
    s++;
  if (*s != '\0') 
    {
      *s = '\0';
      *save_ptr = s + 1;
    }
  else 
    *save_ptr = s;
  return token;
}
```

可以看到在这个函数里面，第一个是待处理字符串，第二个是分隔符号，第三个是需要被存储的字符串。然后我们就完成了arguments的分割，理论上说我们的任务应该就完成，但是程序依然跑不起来。

我又仔细看了一下代码，发现还有一个程序我没有分析，就是thread\_create参数里填的process\_start，根据上一个project的理解，这里是要执行这个函数，所以我进入了这个函数。发现我们其实需要在启动进程的时候把参数全部放进申请号的栈空间里。但是如何放呢？Pintos手册给出了方案。

Pintos手册中给出的process参数在内存里的情况中（默认初始地址为0xbfffffcc）可以看到所有的参数分成了两部分存放，一个是名字一个是地址，在最低的地址上存放返回值。这让我想到了上个实验里Pintos所说的东西，在电脑的内存里，内核是占用PHYS\_BASE到4GB的内存空间，而用户使用0到PHYS\_BASE的空间，并且用户栈是向下生长的，数据段是向上生长的。这样一来，等到要调用数据的时候，指针就应该是从下至上来进行读取。因而压栈的时候就是相反的顺序。首先我们需要把函数的参数名压栈。

那么现在我们就需要来实现这个压栈的过程。于是有如下函数：

```
void
push_argument (void **esp, int argc, int argv[]){
  *esp = (int)*esp & 0xfffffffc;
  *esp -= 4;
  *(int *) *esp = 0;

  for (int i = argc - 1; i >= 0; i--)
  {
    *esp -= 4;
    *(int *) *esp = argv[i];
  }
  *esp -= 4;
  *(int *) *esp = (int) *esp + 4;//压入argv[0]的地址
  *esp -= 4;
  *(int *) *esp = argc;
  *esp -= 4;
  *(int *) *esp = 0;
}
```

可以看到，在这个函数初始了最开始的栈地址。然后进行了参数地址的压栈。通过这个函数可以把已知的已被压栈的一组参数名的地址压入栈内存中。

经过上述操作后，第一个任务基本就算是基本完成了，至此任务一结束。

#### 实验结果

因为这个时候还没有实现后面的syscall部分，所以没有办法进行直观的展示，所以本部分的实验结果先不进行展示。

#### 问题及解决办法

我做完project1后再做project2时发现不能够开启运行磁盘，然后发现原来是我没有创建磁盘。而在创建磁盘后依然出现了代码不能运行的问题，最后发现是userprog文件夹里面的Makefile.var文件必须重新配置，在配置完后就可以开始做了。

另外一件事，就是在刚开始的过程中会出现page fault，原因是因为我在上一个project中实现sema的时候会使用schedule。这样一来，系统启动的时候在还没有分配页的时候就会在sema的信号量操作中进行schedule，进一步出发断言的错误。改正的方法是，加入一个全局变量READY_RUN，一开始为false，当runtask初始化完毕以后再设置成ture，而对于信号量操作，在READY_RUN等于true的时候才能够使用schedule。

### 任务2：Process Control Syscalls

#### 任务描述

当前的Pintos不支持任何的系统调用，我需要再添加如下的系统调用：exit、halt、exec、wait。每个系统调用都有在syscall.c中用户线程级别的对应函数。

exit系统调用是直接退出；halt的作用是关闭系统；exec系统调用会使用函数process\_execute；wait系统调用会等待特定的子进程结束退出。

为了实现这些系统调用，我需要使用一个安全的方法来读写内存（用户虚拟内存空间）。我必须避免内核读入空指针而崩溃发生，也就是说即使有内核读到了空指针也不能出现系统崩溃的问题。

我需要解决因为无效内存而出现的系统调用失效的情况。Pintos已经给出了异常指针的几种情况：空指针、无效指针（指向了无效的地址空间）、指向内核空间。当出现了以上情况，我要做的就是关掉对应的用户进程。

#### 实验过程

经过上面对任务的描述，可以知道我需要干的事情就是进一步修改、实现exit、halt、exec、wait这四个函数。

Pintos在这里提示要看一下lib文件下的syscall文件

```
#ifndef __LIB_SYSCALL_NR_H
#define __LIB_SYSCALL_NR_H

/* System call numbers. */
enum 
  {
    /* Projects 2 and later. */
    SYS_HALT,                   /* Halt the operating system. */
    SYS_EXIT,                   /* Terminate this process. */
    SYS_EXEC,                   /* Start another process. */
    SYS_WAIT,                   /* Wait for a child process to die. */
    SYS_CREATE,                 /* Create a file. */
    SYS_REMOVE,                 /* Delete a file. */
    SYS_OPEN,                   /* Open a file. */
    SYS_FILESIZE,               /* Obtain a file's size. */
    SYS_READ,                   /* Read from a file. */
    SYS_WRITE,                  /* Write to a file. */
    SYS_SEEK,                   /* Change position in a file. */
    SYS_TELL,                   /* Report current position in a file. */
    SYS_CLOSE,                  /* Close a file. */

    /* Project 3 and optionally project 4. */
    SYS_MMAP,                   /* Map a file into memory. */
    SYS_MUNMAP,                 /* Remove a memory mapping. */

    /* Project 4 only. */
    SYS_CHDIR,                  /* Change the current directory. */
    SYS_MKDIR,                  /* Create a directory. */
    SYS_READDIR,                /* Reads a directory entry. */
    SYS_ISDIR,                  /* Tests if a fd represents a directory. */
    SYS_INUMBER                 /* Returns the inode number for a fd. */
  };

#endif /* lib/syscall-nr.h */

```

可以看到第7行到20行就是我的任务了。然后我再回到syscall.c和syscall.h，发现源代码空无一物。

当前只在syscall.c中给了syscall\_init函数和syscall\_handler函数。前者是初始化系统调用，当在用户产生中断的时候进行系统调用的检查并给出相应的操作。

在刚刚的任务一里，我已经完成了数据的压栈工作，现在在这个任务里，和之前相似，我需要从相关的栈中取数据，拿到参数交给syscall\_handler，然后通过syscall\_init中来判断是否有这个操作，最后进行中断、系统调用。

现在我们先来实现最基本的东西，就是所有操作的前提条件——指针要合法。不合法的指针上面已经说过了，空指针、无效指针、指向内核的指针。

以上为实现方法，要注意的是get\_user这个方法在霍普金斯大学的官方代码中是有提供的，所以我使用了这部分代码，其作用是判断该地址是否写入内核了。所以逻辑就变得很简单了，70行是判断是否地址属于用户地址，75行是判断分配的页地址是否有被映射，低84行是判断是否也地址进入了内核的地址范围。

另外，Pintos手册提示中提到，我需要在用户指针导致pagefault的时候在page\_fault函数中把eip寄存器中置为下一个函数的入口，eax设置为-1。所以我在函数中添加了如下代码：

```
void * 
check_ptr2(const void *vaddr)
{ 
  /* Judge address */
  if (!is_user_vaddr(vaddr))//是否为用户地址
  {
    exit_special ();
  }
  /* Judge the page */
  void *ptr = pagedir_get_page (thread_current()->pagedir, vaddr);//是否为用户地址
  if (!ptr)
  {
    exit_special ();
  }
  /* Judge the content of page */
  uint8_t *check_byteptr = (uint8_t *) vaddr;
  for (uint8_t i = 0; i < 4; i++) 
  {
    if (get_user(check_byteptr + i) == -1)
    {
      exit_special ();
    }
  }

  return ptr;
}
```

接下来就轮到具体系统调用的实现了。首先定义一下我们要实现的三个操作。

sys\_halt函数是相对好实现的一个函数，在这个函数里我只需要把进程停掉就行了，实现如下：

```
sys_halt (struct intr_frame* f)
{
  shutdown_power_off();
}
```

下面是sys\_exec函数的实现，首先在156行获得函数的地址，然后在157行、158行检查得到的指针所包含参数以及参数地址指向地址的合法性，之后执行process\_execute，返回值赋值给eax（记录执行结果）。

```
void 
sys_exec (struct intr_frame* f)
{
  uint32_t *user_ptr = f->esp;
  check_ptr2 (user_ptr + 1);//检查第一个参数的地址
  check_ptr2 (*(user_ptr + 1));//检查第一个参数的值，即const char *file指向的地址
  *user_ptr++;
  f->eax = process_execute((char*)* user_ptr);//使用process_execute完成pid的返回
}
```

接下来是sys\_exit函数的实现，和上面基本一样，先获得esp，然后检查参数参数的合法性，最后标记当前线程的st\_exit变量（保存退出状态）然后退出。

```
void 
sys_exit (struct intr_frame* f)
{
  uint32_t *user_ptr = f->esp;
  check_ptr2 (user_ptr + 1);//检验第一个参数
  *user_ptr++;//指针指向第一个参数
  /* record the exit status of the process */
  thread_current()->st_exit = *user_ptr;//保存exit_code
  thread_exit ();
}
```

sys\_wait函数的实现逻辑和上面完全一样，如下所示：

```
void 
sys_wait (struct intr_frame* f)
{
  uint32_t *user_ptr = f->esp;
  check_ptr2 (user_ptr + 1);
  *user_ptr++;
  f->eax = process_wait(*user_ptr);
}
```

但是这个里面我使用的process\_wait是一个还没有实现的函数。

我们需要等待一个特定线程号直到它死掉然后返回它的退出状态。如果它被内核关闭了那么返回-1，如果线程号无效或者不是一个的子进程，又或者这个函数已经执行完了，那就不再等待直接返回-1。

```
Modify Process wait to satisfy some special test in Task1 and also some bugs in other Tasks */
int
process_wait (tid_t child_tid UNUSED)
{
  /* Find the child's ID that the current thread waits for and sema down the child's semaphore */
  struct list *l = &thread_current()->childs;
  struct list_elem *child_elem_ptr;
  child_elem_ptr = list_begin (l);
  struct child *child_ptr = NULL;
  while (child_elem_ptr != list_end (l))//遍历当前线程的所有子线程
  {
    /* list_entry:Converts pointer to list element LIST_ELEM into a pointer to
   the structure that LIST_ELEM is embedded inside.  Supply the
   name of the outer structure STRUCT and the member name MEMBER
   of the list element. */
    child_ptr = list_entry (child_elem_ptr, struct child, child_elem);//把child_elem的指针变成child的指针
    if (child_ptr->tid == child_tid)//找到child_tid
    {
      if (!child_ptr->isrun)//检查子线程之前是否已经等待过
      {
        child_ptr->isrun = true;
        sema_down (&child_ptr->sema);//线程阻塞，等待子进程结束
        break;
      } 
      else //等待过了，has already been successfully called for the given TID
      {
        return -1;
      }
    }
    child_elem_ptr = list_next (child_elem_ptr);
  }
  if (child_elem_ptr == list_end (l)) {//找不到child_tid
    return -1;
  }
  //执行到这里说明子进程正常退出
  list_remove (child_elem_ptr);//从子进程列表中删除该子进程，因为它已经没有在运行了，也就是说父进程重新抢占回了资源
  return child_ptr->store_exit;//返回子线程exit值
}
```

现在对上面的代码进行讲解，175行的时候获得需要等待的进程的子进程列表，然后再179行的时候开始遍历，如果遍历的时候找到了子节点，那么就进行判断，如果子线程没有运行过（188行，isrun表示是否运行过），那么就让子线程完成，然后进行V操作来阻塞当前线程，直到子线程完成，最后break返回其退出的状态；如果子线程运行过了（194行），那么就直接返回-1；又或者执行process\_wait操作的进程没有子线程（201行），那么就直接返回-1；

上面整体上的逻辑其实并没有内存部分绕。至此，任务二结束。

#### 实验结果

在写完任务二时我忘记给make check进行截图了。所以这里没有对应的make check的截图。

#### 问题及解决办法

本任务最大的问题就是栈指针的形式不清楚，这导致我在写的时候出现了错误，然后调试的时候发生page\_fault错误，最简单的例子就是esp的使用，把\*（int \*）esp写成了\*esp。解决办法是：在我上GitHub参考其他Pintos实现代码的时候发现自己出现了错误，然后才把这个错误改正过来。

### 任务3：File Operation Syscalls

#### 任务描述

在上一个任务中我已经完成几个进程系统调用，在这个任务中，Pintos手册又要求实现另一部分系统调用，这部分是文件系统调用。

Pintos的文件系统目前不是线程安全的，我必须确保文件系统调用不会同时调用多个文件系统调用（多个线程同时对一个文件进行操作）。需要实现的文件系统调用已经有在文件中有体现。

#### 实验过程

首先来看一下我们需要实现的函数，来到syscall-nr.h，当前project2部分还有这些函数没有实现。

需要实现的函数知道了，那么该怎么实现呢？再次看Pintos手册，这些关键词给出了我们要做的事情——&quot;thread-safe&quot;、&quot;do not call multiple file system functions concurrently&quot;。所以我要做的其实也不是很难，就是实现的时候加锁就好了。

然后进一步思考，一个系统调用是不是可以操作多个文件，答案显然是&quot;可以的&quot;。那我现在需要一个方法记录下一个当前线程所使用的文件，为了使后面程序调用更加灵活，这里定义两个与文件相关的代码块，一个是单独记录文件使用情况的thread\_files，另一个是thread中记录自身文件的files.

fd代表文件描述符。Linux中其被分为三种，标准输入、标准输出、标准错误，对应的值分别为0、1、2。现在再用最简单的方法定义一下锁的释放和获取。

先分析一下锁该怎么进行操作。对于文件的锁，在调用与文件相关的系统调用时，会进行锁的获取，当结束调用的时候会释放锁。所以可以发现，不论如何，我应该在线程退出的时候进行文件锁的释放。

确定好这些之后，就开始依次实现了，考虑到参数传递部分需要write，所以这里先实现write。在这个部分Pintos手册很温馨地给出了各个函数的参数形式还有返回值，这无疑减轻了许多难度。

在Pintos手册上对write的函数定义如下：

```
int write(int fd,const void*buffer,unsigned size);
```

对于返回值规定为：如果读取到了数据，返回读入的buffer的长度，如果没有写入返回0。而我所读到的所有buffer应当使用putbuf呈现在控制台上。

```
void 
sys_write (struct intr_frame* f)
{
  uint32_t *user_ptr = f->esp;
  check_ptr2 (user_ptr + 7);//for tests maybe?
  check_ptr2 (*(user_ptr + 6));
  *user_ptr++;
  int fd = *user_ptr;
  const char * buffer = (const char *)*(user_ptr+1);
  off_t size = *(user_ptr+2);
  if (fd == 1) {//writes to the console
    /* Use putbuf to do testing */
    putbuf(buffer,size);
    f->eax = size;//return number written
  }
  else
  {
    /* Write to Files */
    struct thread_file * thread_file_temp = find_file_id (*user_ptr);
    if (thread_file_temp)
    {
      acquire_lock_f ();//file operating needs lock
      f->eax = file_write (thread_file_temp->file, buffer, size);
      release_lock_f ();
    } 
    else
    {
      f->eax = 0;//can't write,return 0
    }
  }
}

```

在这个函数里我没有用原始的模型，并且使用了一个file.c中的函数file\_write来帮助我完成任务。

这个函数能帮助我完成文件的写入，所以我在sys\_write中做的事就是把这三个参数送到这个函数里。首先我先获得中断的页，然后通过esp获得数据的栈顶的地址（183行），然后我判断一下获得的esp以及esp所指向地址的合法性（184行和185行），如果是标准输入（fd==1），那么使用putbuf进行输出。否则fd就是0或者2，这个时候直接就对文件列表搜索，如果出现了对应id的标识就进行输入。其中find\_file\_id函数实现如下，实现逻辑就是遍历判断id：

```
struct thread_file * 
find_file_id (int file_id)
{
  struct list_elem *e;
  struct thread_file * thread_file_temp = NULL;
  struct list *files = &thread_current ()->files;
  for (e = list_begin (files); e != list_end (files); e = list_next (e)){
    thread_file_temp = list_entry (e, struct thread_file, file_elem);
    if (file_id == thread_file_temp->fd)
      return thread_file_temp;
  }
  return false;
}

```

经过上面的操作后，write就写完了。接下来是create，Pintos指南的定义如下：

```
bool create(const char*file,unsigned initial_size);
```

其中file为文件名，initial\_size为初始定义大小。这个函数是比较好实现的，这是因为在这个时候我在filesys文件夹下面发现了很多我用得上的函数，在这个函数里，我可以用到filesys\_create，这个是代码框架已经实现的函数。

可以看到，这就是我们需要实现的函数，那么我只需要传参数就好了。实现如下：

```
void 
sys_create(struct intr_frame* f)
{
  uint32_t *user_ptr = f->esp;
  check_ptr2 (user_ptr + 5);//for tests maybe?
  check_ptr2 (*(user_ptr + 4));
  *user_ptr++;
  acquire_lock_f ();
  f->eax = filesys_create ((const char *)*user_ptr, *(user_ptr+1));
  release_lock_f ();
}

```

这个函数逻辑很简单，就直接先对栈寄存器指针和所指向的地址进行合法性检查，然后传参数就行（235行是为了跳过返回地址，237行里，\*user\_ptr是第一个参数，\*（user\_ptr+1）为第二个参数，关于参数的知识，在第一个任务已经讲过）。

接下来是remove操作，Pintos手册给出的函数声明如下：

```
bool remove(const char* file);
```

这里的操作和上面的create基本一样，file.c有源代码可以用，实现如下所示：

```
void 
sys_remove(struct intr_frame* f)
{
  uint32_t *user_ptr = f->esp;
  check_ptr2 (user_ptr + 1);//arg address
  check_ptr2 (*(user_ptr + 1));//file address 
  *user_ptr++;
  acquire_lock_f ();
  f->eax = filesys_remove ((const char *)*user_ptr);
  release_lock_f ();
}
```

然后是open操作，Pintos手册给出的声明如下：

```
int open(const char* file);
```

提示是这样说的，打开一个文件，如果打开成功就返回一个非负的fd值，否则返回-1。另外每个进程所拥有的fd值是独立的，也就是子进程不会从父进程中继承fd值（fd值是什么我在上文已经提到，这里不再赘述），另外需要注意的事情是，当一个文件被打开多次，不管打开的进程是不是同一个进程，都返回一个新的fd值（意思大概是一个进程可能会重复操作一个文件，比如说process1打开了文件A，过一会他又打开了文件A，这个时候process1还需要调用一次open，在上文我已经说过，我们的数据结构会对process操作的file进行维护，依次加入队列，那么现在的意思就是，无论队列之前有没有对该文件做过操作的记录，都要再做一次记录然后返回）。实现如下：

```
void 
sys_open (struct intr_frame* f)
{
  uint32_t *user_ptr = f->esp;
  check_ptr2 (user_ptr + 1);
  check_ptr2 (*(user_ptr + 1));
  *user_ptr++;
  acquire_lock_f ();
  struct file * file_opened = filesys_open((const char *)*user_ptr);
  release_lock_f ();
  struct thread * t = thread_current();
  if (file_opened)
  {
    struct thread_file *thread_file_temp = malloc(sizeof(struct thread_file));
    thread_file_temp->fd = t->max_file_fd++;
    thread_file_temp->file = file_opened;
    list_push_back (&t->files, &thread_file_temp->file_elem);//维护files列表
    f->eax = thread_file_temp->fd;
  } 
  else// the file could not be opened
  {
    f->eax = -1;
  }
}
```

前面几行的操作的意思不在解释，上文已经解释过几次了，同样的file.c又源代码可以用，位filesys\_open，然后再266行开始对fd值进行维护，274行是考虑打开失败的情况。

然后是filesize，声明如下：

```
int filesize(int fd);
```

这个函数思路比较简单，就是用在现有的文件链表里面id值位fd的文件，然后调用file.c里的file\_length函数就好了，然后把结果值放在eax里面，当然如果没有找到对应的文件就返回-1（把eax赋值-1）即可。实现如下：

```
void 
sys_filesize (struct intr_frame* f){
  uint32_t *user_ptr = f->esp;
  check_ptr2 (user_ptr + 1);
  *user_ptr++;//fd
  struct thread_file * thread_file_temp = find_file_id (*user_ptr);
  if (thread_file_temp)
  {
    acquire_lock_f ();
    f->eax = file_length (thread_file_temp->file);//return the size in bytes
    release_lock_f ();
  } 
  else
  {
    f->eax = -1;
  }
}
```

再之后就是read函数了，Pintos手册上声明如下：

```
int read(int fd,void* buffer,unsigned size);
```

图4-3-18 read函数的声明

提示是说如果通过fd来判断是否读入，size代表buffer大小，最后返回操作的字符大小（无法操作返回-1）。跟上面的思路基本一样，代码给出如下：

```
void 
sys_read (struct intr_frame* f)
{
  uint32_t *user_ptr = f->esp;
  /* PASS the test bad read */
  *user_ptr++;
  /* We don't konw how to fix the bug, just check the pointer */
  int fd = *user_ptr;
  uint8_t * buffer = (uint8_t*)*(user_ptr+1);
  off_t size = *(user_ptr+2);
  if (!is_valid_pointer (buffer, 1) || !is_valid_pointer (buffer + size,1)){
    exit_special ();
  }
  /* get the files buffer */
  if (fd == 0) //stdin
  {
    for (int i = 0; i < size; i++)
      buffer[i] = input_getc();
    f->eax = size;
  }
  else
  {
    struct thread_file * thread_file_temp = find_file_id (*user_ptr);
    if (thread_file_temp)
    {
      acquire_lock_f ();
      f->eax = file_read (thread_file_temp->file, buffer, size);
      release_lock_f ();
    } 
    else//can't read
    {
      f->eax = -1;
    }
  }
}

```

之后的函数基本都差不多了，接下来是seek函数，声明如下：

```
void seek(int fd,unsigned position);
```

这个函数的意思是把fd这个文件下一次需要操作的位置改为positon，实现如下：

```
void 
sys_seek(struct intr_frame* f)
{
  uint32_t *user_ptr = f->esp;
  check_ptr2 (user_ptr + 5);
  *user_ptr++;//fd
  struct thread_file *file_temp = find_file_id (*user_ptr);
  if (file_temp)
  {
    acquire_lock_f ();
    file_seek (file_temp->file, *(user_ptr+1));
    release_lock_f ();
  }
}

```

然后是tell函数，意思刚好和seek相反，它的含义是要返回下一个要操作的position，只是我不需要考虑具体实现逻辑，只需要调用file.c里面的file\_tell就好了，定义和实现一并给出：

```
unsigned tell(int fd);
```

```
void 
sys_tell (struct intr_frame* f)
{
  uint32_t *user_ptr = f->esp;
  check_ptr2 (user_ptr + 1);
  *user_ptr++;
  struct thread_file *thread_file_temp = find_file_id (*user_ptr);
  if (thread_file_temp)
  {
    acquire_lock_f ();
    f->eax = file_tell (thread_file_temp->file);
    release_lock_f ();
  }else{
    f->eax = -1;
  }
}
```

最后一个函数是close函数，这个函数会为一个文件操作进行收尾。定义如下：

```
void close(int fd);
```

这个函数的意思是关闭相应标号的文件，操作逻辑应该是简单的，搜索文件列表，fd相等，那么关闭并从之前维护的列表里remove掉。

```
void 
sys_close (struct intr_frame* f)
{
  uint32_t *user_ptr = f->esp;
  check_ptr2 (user_ptr + 1);
  *user_ptr++;
  struct thread_file * opened_file = find_file_id (*user_ptr);
  if (opened_file)
  {
    acquire_lock_f ();
    file_close (opened_file->file);
    release_lock_f ();
    /* Remove the opened file from the list */
    list_remove (&opened_file->file_elem);
    /* Free opened files */
    free (opened_file);
  }
}
```

至此，80分版本的任务3完成。

正如我如上所说，这是80分版本的结束，对于2021年本版，满分是90分，所以还多了一些任务。其中其中有一个函数为practice，作用是返回第一个参数加一。

之后就是今年版本特有的检查点，浮点数了，对应前缀为fp的测试点，很遗憾的是，到提交这份报告的时候我仍然又三个点过不去，关于这一部分的实现，我会在问题及解决中给出。

#### 实验结果

最后的线上平台测试结果为93分。

#### 问题及解决办法

通过查看今年官网上最新Pintos的内容，发现今年多了一些测试点，所以我复制了tests文件夹下的userprog模块，重新进行了make check。发现出现了错误，原因是在新的测试点中出现了新的系统调用。

再次查看Pintos手册，发现了新的系统调用practice。这个系统调用的声明为：
```
int practice(int i);
```
对于practice，描述是，这是一个不存在的函数调用，在现代的操作系统中并没有这样一个系统调用。它的作用时把int型的参数加一然后返回它。实现如下：

```
void 
sys_practice (struct intr_frame *f)
{
  uint32_t *ptr = f->esp;
  int n = *(int *)check_ptr (++ptr);
  int ret_val;

  ret_val = n + 1;

  f->eax = ret_val;
}
```

完成这个之后有一部分的测试点可以通过了，这次的成绩时90。

接下来我翻阅了2021fall的Pintos文档，发现还有一个浮点数的测试。所以接下来将讲述如何实现浮点数。

文档里这样描述：
```
The Pintos kernel does not currently support Foating point operations. You must update the OS so that
both user programs and the kernel can use Foating point instructions.
This may seem like a daunting task, but rest assurred, the OS doesn't have to do much when it comes
to implementing Foating point instructions: the compiler does the hard work of generating Foating point
instructions, and the hardware does the hard work of executing Foating point instructions. However, because
there is only one Foating-point unit1
(FPU) on the CPU, all threads on must share it. This is where the OS
comes into play. The OS must:
1. Save the state of the FPU on context switches
2. Save the state of the FPU on interrupts & system calls
```
可以看到，大致意思就是说，现在的Pintos不支持浮点操作，我必须更新代码来实现浮点操作。

对于CPU来说，只有一个专门的单元来操作浮点，也就是FPU，所以所有的线程都需要共享这个存储器，那么我现在要做的就是如下的两件事：
- 1、保存内容转换时FPU的状态。
- 2、保存中断和系统调用时FPU的状态。
该怎么实现文档也说了。
```
• Update one line of src/threads/start.S to allow FPU instructions to be executed on the hardware
(see 4.2.2 Enabling the FPU)
• Ensure Foating-point registers are initialized correctly when a new thread or process is created (see
4.2.3 FPU Initialization), unlike with GPRs
• Implement a system call to compute the value e (see 4.2.4 FPU Syscall)
```
接下来是文档中的步骤：
```
You must update this as follows:
- orl $CR0_PE | CR0_PG | CR0_WP | CR0_EM, %eax
+ orl $CR0_PE | CR0_PG | CR0_WP, %eax
in src/threads/start.S to remove the flag and indicate to the processor that the FPU is present.
```
首先到start.S文件夹下面，把上面的部分更新，告诉调度器现在浮点数是可以存在的。

之后文档的描述如下：

```
you must ensure:
1. Initialize the FPU at OS startup
2. Re-initialize the FPU when a new thread is created
3. Re-initialize the FPU when a new process is created
```

意思是我必须确保Pintos启动的时候FPU是打开的，然后再开启新线程和新进程的时候重新初始化FPU。当然，在初始化的时候也必须把之前的FPU保存起来，这也是这个任务的目标。

然后接下来手册讲了一下需要使用汇编来直接操作寄存器。具体怎么操作？手册让我自己看代码。好吧，那我们开始吧。

通过之前的学习，我已经明白当没有头绪的时候看测试用例就是最好的办法。然后我研究了一下测试用例，发现了下面的几种汇编的用法，通过网上的查询，asm volatile是用来调用汇编指令的。我发现了这样两种用法，那我把这两个东西搞明白应该就好了。

```
# init fpu
    fninit
    
asm("fsave (%0); fninit; fsave (%1)" : : "g"(&fpu), "g"(&init_fpu));

/* Invokes syscall NUMBER, passing argument ARG0, and returns the
   return value as an `int'. */
asm volatile("pushl %[arg0]; pushl %[number]; int $0x30; addl $8, %%esp"                       \
                 : "=a"(retval)                                                                    \
                 : [number] "i"(NUMBER), [arg0] "g"(ARG0)                                          \
                 : "memory");                                                                      \
```

说实话，看完这段指令之后很懵，我知道这是汇编调用，但是具体格式确实不是很了解，所以之后照猫画虎。在网上，我找到了FPU中我需要的两个指令，如下所示。

```
FSAVE dest  Store FPU state: store FPU state into 94-bytes at dest
FRSTOR src  Load FPU state: restore FPU state as saved by FSAVE
```

然后结合测试用例里面的写法我做出了如下的改进。
按照要求所说的，创建的时候先保存再初始化，所以在thread_create和schedule中加入如下代码：

```
  asm volatile ("fsave %0"::"m"(cur_t->fpu_state[0]));
  asm volatile ("fninit");
  asm volatile ("fsave %0"::"m"(t->fpu_state[0]));
  asm volatile ("frstor %0"::"m"(cur_t->fpu_state[0]));
```

当然，fpu_state是我新加入的成员变量，再thread的结构体中定义：

```
uint8_t fpu_state[FPU_SIZE];
```

然后再次进行make check，这次有93个pass，但是还是有三个测试点没有通过。其实我还挺庆幸的，因为这个汇编代码确实看的我有点头疼。

因为时间原因，剩下的三个测试点我选择了放弃，至此任务三以一点遗憾告终。

最终的线上成绩是94。


