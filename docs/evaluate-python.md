---
layout: default
---

_cProfile_ and _profile_ provide deterministic profiling of Python programs. A profile is a set of statistics that describes how often and for how long various parts of the program executed. These statistics can be formatted into reports via the pstats module. According to [offical documentation](https://docs.python.org/3/library/profile.html)

**cProfile**：基于lsprof的用C语言实现的扩展应用，运行开销比较合理，适合分析运行时间较长的程序，推荐使用这个模块.
**profile**：纯Python实现的性能分析模块，接口和cProfile一致。但在分析程序时增加了很大的运行开销。如果想扩展profiler的功能，可以通过继承这个模块实现.

* * *

## Evaluate with cProfile

### Example code

```python
import cProfile
import pstats

def fib(n):
    if n == 0:
        return 0
    elif n == 1:
        return 1
    else:
        return fib(n-1) + fib(n-2)

def fib_seq(n):
    seq = []
    if n > 0:
        seq.extend(fib_seq(n-1))
    seq.append(fib(n))
    return seq

# output to console 
python -m cProfile test.py

# output to file "restats"
cProfile.run('fib_seq(20)', filename="restats", sort="cumulative")

# record all the infomation and print to console
pr = cProfile.Profile()
pr.enable()
fib_seq(20)
pr.disable()
# pr.print_stats()
s = io.StringIO()
ps = pstats.Stats(pr, stream=s).sort_stats('cumulative')
ps.print_stats()
```

### Output
```
/usr/local/bin/python3.6 /Users/meteor/PycharmProjects/testProj/main.py
================================================================================
Wed May 22 11:29:39 2019    profile_stats_0.stats
Wed May 22 11:29:39 2019    profile_stats_1.stats
Wed May 22 11:29:39 2019    profile_stats_2.stats
Wed May 22 11:29:39 2019    profile_stats_3.stats
Wed May 22 11:29:39 2019    profile_stats_4.stats

         286780 function calls (330 primitive calls) in 0.109 seconds

   Ordered by: cumulative time

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        5    0.001    0.000    0.109    0.022 {built-in method builtins.exec}
        5    0.000    0.000    0.108    0.022 <string>:1(<module>)
    105/5    0.000    0.000    0.108    0.022 main.py:13(fib_seq)
286455/105    0.108    0.000    0.108    0.001 main.py:5(fib)
      100    0.000    0.000    0.000    0.000 {method 'extend' of 'list' objects}
      105    0.000    0.000    0.000    0.000 {method 'append' of 'list' objects}
        5    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}

```
> It takes 286780 separate function calls (330 primitive calls while the rest is recursive) totally in 0.109 seconds.


#### Description and sort key

| name         | head two          | sort key      |
|:-------------|:------------------|:------|
| ncalls       | number of calls, the second value is the number of primitive calls and the former is the total number of calls | 'calls'  |
| tottime      | total time spent in a function   | 'time'  |
| percall      | the quotient of tottime divided by ncalls      |    |
| cumtime      | cumulative time spent in a function | 'cumulative'  |
| percall      | the quotient of cumtime divided by primitive calls      |    |



## Evaluate with pstats

### Example code

```python

# pstats.Stats类是一个可以读取cProfile输出的文件，组合排序，输出整理的结果
# Create 5 set of stats
filenames = []
for i in range(5):
    filename = 'profile_stats_%d.stats' % i
    cProfile.run('fib_seq(20)', filename)

# Read all 5 stats files into a single object
stats = pstats.Stats('profile_stats_0.stats')
for i in range(1, 5):
    stats.add('profile_stats_%d.stats' % i)

# Clean up filenames for the report (remove the extraneous directory names)
stats.strip_dirs()

# Sort the statistics by the cumulative time spent in the function
stats.sort_stats('cumulative')

# .5 means output only 50% lines of the result
stats.print_stats(.5, '\(fib')

# # limit output to lines with "(fib" in them
# stats.print_stats('\(fib')
# # limit only 5 output lines
# stats.print_stats(5)
```
