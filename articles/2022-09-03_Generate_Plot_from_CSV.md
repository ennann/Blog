# Background

Lately I got a requirement that need to generate a plot from a CSV file. Each line represents a data point, and the first column is the x-axis, and the other column is the y-axis. The plot should be generated in a PDF file. I searched the internet and found that there are many ways to do this. I will introduce how I did it.

# Data Format

The data was stored in a csv file as below. The first line represents the different examinations at different times, and the following line represents the ranking information at the different time points.

```
姓名,7上 期中,7上 12月,7上 期末,7下 3月,7下 期中,7下 5月,7下 期末
胡桐榽,19,24,21,41,15,22,14
刘忆天,38,81,23,15,41,43,21
罗亦琪,69,44,33,34,64,47,24
...

```

Clearly, the first line needs to be the X-axis, and the first element in the first line is useless for a plot. So I just need the second element to the last. But the following lines are different. I need to use the first element as the title of the plot, and the rest of it as the Y-axis.

# Thinking Through

The first thing is to read the csv file to the program. Since I am using Python as my programming language, I choose to read the file directly. The code will be as this

```python
def read_csv(filename):
	"""
	:param filename:
	:return: a list of lists that contains each line in the csv file    
	"""
	with open(filename, 'r') as f:
	    lines = f.readlines()
	    lines = [line.strip() for line in lines]
		  lines = [line.split(',') for line in lines]
	    return lines
```

This code will read the csv file and return a list that contains each line's data and

```python
[
['姓名', '7上 期中', '7上 12月', '7上 期末', '7下 3月', '7下 期中', '7下 5月', '7下 期末'],
['胡桐榽', '19', '24', '21', '41', '15', '22', '14'],
['刘忆天', '38', '81', '23', '15', '41', '43', '21'],
['罗亦琪', '69', '44', '33', '34', '64', '47', '24']
]

```

Because it's in a list, I need to use *`for`* loop.

```python
for i in range(1, len(csv_data)):
    title = csv_data[i][0]
    x_axis = csv_data[0][1:]
    y_axis = csv_data[i][1:]
    y_axis = [float(y) for y in y_axis]
    df = pd.DataFrame({'时间': x_axis, '成绩': y_axis}) # generate a DataFrame
    df.loc[df['成绩'] == 0, '成绩'] = np.nan  # replace 0 to NaN (numpy)

```

Here I finished the data processing, and now its time to generate plot.

```coffeescript
plt.figure(figsize=(10, 6), dpi=200)
plt.plot(df['时间'], df['成绩'], color=(0.41, 0.74, 0.97), linewidth=2.0, linestyle='--', marker='o', markerfacecolor=(0.49, 0.89, 0.60), markersize=8)

plt.ylabel('排名变化', fontsize=15)
plt.xticks(fontsize=15)
plt.yticks(fontsize=15)

y_max = max([ele for ele in df['成绩'] if np.isnan(ele) == False])
y_min = min([ele for ele in df['成绩'] if np.isnan(ele) == False])

# 设置坐标轴范围
plt.ylim(y_max + 15, y_min - 30)

# 设置数据标签
for a, b in zip(df['时间'], df['成绩']):
    if b == np.nan: continue  # 跳过 NaN 值
    plt.text(a, b - (y_max - y_min) * 0.1, '%.0f' % b, va='bottom', ha='center', fontsize=10,
             bbox=dict(boxstyle='round,pad=0.5', facecolor=(0.98, 0.83, 0.52), edgecolor='k', lw=1, alpha=0.5))

plt.title(csv_data[i][0], fontsize=22)
plt.savefig('./data/' + str(i) + ' ' + csv_data[i][0] + '.png')
plt.close()

```

# Final Work

The final code is as follows. Also, you can check the example picture in the end.

```python
import matplotlib
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
matplotlib.rc("font", family='OPPOSans')

# 读取 csv 函数
def read_csv(filename):
""":param filename::return: a list of lists that contains each line in the csv file    """with open(filename, 'r') as f:
        lines = f.readlines()
        lines = [line.strip() for line in lines]
        lines = [line.split(',') for line in lines]
        return lines

csv_data = read_csv('/Users/Elton/Downloads/data/data.csv')

x_plot = csv_data[0][1:]

for i in range(1, len(csv_data)):
    title = csv_data[i][0]
    x_axis = csv_data[0][1:]
    y_axis = csv_data[i][1:]
    y_axis = [float(y) for y in y_axis]
    df = pd.DataFrame({'时间': x_axis, '成绩': y_axis}) # generate a DataFrame
    df.loc[df['成绩'] == 0, '成绩'] = np.nan  # replace 0 to NaN (numpy)

    plt.figure(figsize=(10, 6), dpi=200)
    plt.plot(df['时间'], df['成绩'], color=(0.41, 0.74, 0.97), linewidth=2.0, linestyle='--', marker='o', markerfacecolor=(0.49, 0.89, 0.60), markersize=8)

    plt.ylabel('排名变化', fontsize=15)
    plt.xticks(fontsize=15)
    plt.yticks(fontsize=15)

    y_max = max([ele for ele in df['成绩'] if np.isnan(ele) == False])
    y_min = min([ele for ele in df['成绩'] if np.isnan(ele) == False])

    # 设置坐标轴范围
    plt.ylim(y_max + 15, y_min - 30)

    # 设置数据标签
    for a, b in zip(df['时间'], df['成绩']):
        if b == np.nan: continue  # 跳过 NaN 值
        plt.text(a, b - (y_max - y_min) * 0.1, '%.0f' % b, va='bottom', ha='center', fontsize=10,
                 bbox=dict(boxstyle='round,pad=0.5', facecolor=(0.98, 0.83, 0.52), edgecolor='k', lw=1, alpha=0.5))

    plt.title(csv_data[i][0], fontsize=22)
    plt.savefig('./data/' + str(i) + ' ' + csv_data[i][0] + '.png')
    plt.close()

```

![Untitled](Mathplotlib%20Hands%20on%207d0a43fbd33d4933a8dd60183c608855/Untitled.png)

# Problems

When I generated the plot, I found that the Chinese font was a black square. I googled it and

Found that it is utf-8 error. Either you change the mathplotlib config or import the font that already exists in the system. I choose the last solution.

First check all the fonts in the system, code as follow:

```python
from matplotlib.font_manager import FontManager

mpl_fonts = set(f.name for f in FontManager().ttflist)

print('all font list get from matplotlib.font_manager:')
for f in sorted(mpl_fonts):
    print('\t' + f)

```

Then choose the font you want.

```python
matplotlib.rc("font", family='OPPOSans')
```

# References

1. y 轴未按照顺序显示
   1. [https://www.cxyzjd.com/article/weixin_45464159/105516537](https://www.cxyzjd.com/article/weixin_45464159/105516537)
2. 不显示中文
   1. [https://zhuanlan.zhihu.com/p/104081310](
