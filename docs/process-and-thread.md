---
layout: default
---

## What is process and thread
进程是操作系统执行程序时候都一个instance，它可能有一个或者多个线程。通常是开始的时候有一个主线程，主线程会启动其他线程。线程是操作系统调度执行的最小单位（时间片）。

### 进程与线程区别
进程和线程都是操作系统上都特性，不是cpu都特性。进程有着操作系统分配的独立的虚拟地址空间和系统资源（_a virtual address space, executable code, open handles to system objects, a security context, a unique process identifier, environment variables, a priority class, minimum and maximum working set sizes, and at least one thread of execution_），而在同一进程里面的线程是共享存储空间和系统资源的,它们可以同时执行。因为是共享存储，线程间的通信比起进程间的通信简单很多，进程间通信（IPC, Interprocess communication）包括管道, 系统IPC（包括消息队列,信号,共享存储), 套接字(SOCKET)。使用多进程会比较的健壮和安全，但是进程间的切换耗费更高（load数据，读取内存）。

### 为什么使用多进程与多线程
1. 提高效率
2. 防止阻塞。即便是单核cpu,在不提升效率的情况下还是推荐使用多线程。比如经常遇到的场景是请求模型的运算，等待结果返回。这些耗时很长的任务就阻塞主程序往下执行，使用多线程可以实现异步。


## Python中的多进程与多线程
### Multiprocessing with python

```python
from multiprocessing import Process
from os import getpid
from random import randint
from time import time, sleep


def download_task(filename):
    print('启动下载进程，进程号[%d].' % getpid())
    print('开始下载%s...' % filename)
    time_to_download = randint(5, 10)
    sleep(time_to_download)
    print('%s下载完成! 耗费了%d秒' % (filename, time_to_download))


def main():
    start = time()
    p1 = Process(target=download_task, args=('Python从入门到住院.pdf', ))
    p1.start()
    p2 = Process(target=download_task, args=('Peking Hot.avi', ))
    p2.start()
    p1.join()
    p2.join()
    end = time()
    print('总共耗费了%.2f秒.' % (end - start))


if __name__ == '__main__':
    main()
```
上面的代码中，是直接使用新建```Process```对象来实现多进程的。通过```target```参数我们传入一个函数来表示进程启动后要执行的代码，后面的```args```是一个元组，它代表了传递给函数的参数。```Process```对象的start方法用来启动进程，而```join```方法表示等待进程执行结束。


### 进程有独立的地址空间
```
from multiprocessing import Process
from time import sleep

counter = 0


def sub_task(string):
    global counter
    while counter < 10:
        print(string, end='', flush=True)
        counter += 1
        sleep(0.01)

        
def main():
    Process(target=sub_task, args=('Ping', )).start()
    Process(target=sub_task, args=('Pong', )).start()


if __name__ == '__main__':
    main()
```
这段代码的结果是Ping和Pong各输出了10个，因为它们有着各自独立的```counter```变量。


### threading with python

```python

from random import randint
from threading import Thread
from time import time, sleep

def download(filename):
    print('开始下载%s...' % filename)
    time_to_download = randint(5, 10)
    sleep(time_to_download)
    print('%s下载完成! 耗费了%d秒' % (filename, time_to_download))

def main():
    start = time()
    t1 = Thread(target=download, args=('Python从入门到住院.pdf',))
    t1.start()
    t2 = Thread(target=download, args=('Peking Hot.avi',))
    t2.start()
    t1.join()
    t2.join()
    end = time()
    print('总共耗费了%.3f秒' % (end - start))

if __name__ == '__main__':
    main()
```
> 这段代码的结果是直接使用threading模块中的Thread类新建对象实现的，也可以通过继承Thread类的方式来创建自定义的线程类，然后再创建线程对象并启动线程。

```python
from random import randint
from threading import Thread
from time import time, sleep


class DownloadTask(Thread):

    def __init__(self, filename):
        super().__init__()
        self._filename = filename

    def run(self):
        print('开始下载%s...' % self._filename)
        time_to_download = randint(5, 10)
        sleep(time_to_download)
        print('%s下载完成! 耗费了%d秒' % (self._filename, time_to_download))

def main():
    start = time()
    t1 = DownloadTask('Python从入门到住院.pdf')
    t1.start()
    t2 = DownloadTask('Peking Hot.avi')
    t2.start()
    t1.join()
    t2.join()
    end = time()
    print('总共耗费了%.2f秒.' % (end - start))

if __name__ == '__main__':
    main()
```

也可以通过以下线程池的方式实现
#### 1. 简单使用
```python
from concurrent.futures import ThreadPoolExecutor
import time

# 参数times用来模拟网络请求的时间
def get_html(times):
    time.sleep(times)
    print("get page {}s finished".format(times))
    return times

executor = ThreadPoolExecutor(max_workers=2)
# 通过submit函数提交执行的函数到线程池中，submit函数立即返回，不阻塞
task1 = executor.submit(get_html, (3))
task2 = executor.submit(get_html, (2))
# done方法用于判定某个任务是否完成
print(task1.done())
# cancel方法用于取消某个任务,该任务没有放入线程池中才能取消成功
print(task2.cancel())
time.sleep(4)
print(task1.done())
# result方法可以获取task的执行结果
print(task1.result())

# 执行结果
# False  # 表明task1未执行完成
# False  # 表明task2取消失败，因为已经放入了线程池中
# get page 2s finished
# get page 3s finished
# True  # 由于在get page 3s finished之后才打印，所以此时task1必然完成了
# 3     # 得到task1的任务返回值
```
```submit```函数是不阻塞的，会立即返回。可以通过```done```方法判断是否执行完成。但不可能一直通过```done```来判断是否执行完成，所有有下面的代码。


#### 2. as_completed
```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

# 参数times用来模拟网络请求的时间
def get_html(times):
    time.sleep(times)
    print("get page {}s finished".format(times))
    return times

executor = ThreadPoolExecutor(max_workers=2)
urls = [3, 2, 4] # 并不是真的url
all_task = [executor.submit(get_html, (url)) for url in urls]

for future in as_completed(all_task):
    data = future.result()
    print("in main: get page {}s success".format(data))

# 执行结果
# get page 2s finished
# in main: get page 2s success
# get page 3s finished
# in main: get page 3s success
# get page 4s finished
# in main: get page 4s success
```
```as_completed```方法是一个生成器，在没有任务完成的时候，会阻塞，在有某个任务完成的时候，就能执行for循环下面的语句，然后继续阻塞住，循环到所有的任务结束。从结果也可以看出，先完成的任务会先通知主线程（2s的任务先完成）。

#### 3. map
```python
from concurrent.futures import ThreadPoolExecutor
import time

# 参数times用来模拟网络请求的时间
def get_html(times):
    time.sleep(times)
    print("get page {}s finished".format(times))
    return times

executor = ThreadPoolExecutor(max_workers=2)
urls = [3, 2, 4] # 并不是真的url

for data in executor.map(get_html, urls):
    print("in main: get page {}s success".format(data))
# 执行结果
# get page 2s finished
# get page 3s finished
# in main: get page 3s success
# in main: get page 2s success
# get page 4s finished
# in main: get page 4s success
```
使用```map```方法，无需提前使用```submit```方法，```map```方法与python标准库中的map含义相同，都是将序列中的每个元素都执行同一个函数。上面的代码就是对urls的每个元素都执行get_html函数，并分配各线程池。可以看到执行结果与上面的```as_completed```方法的结果不同，输出顺序和urls列表的顺序相同，就算2s的任务先执行完成，也会先打印出3s的任务先完成，再打印2s的任务完成。

也可以通过```with```关键字
```python
import concurrent.futures

# [rest of code]

if __name__ == "__main__":
    with concurrent.futures.ThreadPoolExecutor(max_workers=3) as executor:
        executor.map(thread_function, range(3))
```
可以看到，通过使用线程池，我们不再需要通过给每个线程添加```join```方法来阻塞程序。这样可以防止忘记添加```join```

[Python-100-Days](https://github.com/jackfrued/Python-100-Days/blob/master/Day01-15/13.%E8%BF%9B%E7%A8%8B%E5%92%8C%E7%BA%BF%E7%A8%8B.md)
[ThreadPoolExecutor线程池](https://www.jianshu.com/p/b9b3d66aa0be)
[An Intro to Threading in Python](https://realpython.com/intro-to-python-threading/#using-a-threadpoolexecutor)
