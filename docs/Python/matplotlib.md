# matplotlib

[toc]

```python
import matplotlib.pyplot as plt

if __name__ == '__main__':
    # 绿色线条，红色内填充
    plt.plot([1, 2, 3, 4], [3, 6, 11, 12], marker='o', mfc='red', color='green', linestyle=':')
    # 设置中文字体
    plt.rcParams['font.sans-serif'] = ['SimHei']
    # 设置横纵坐标
    plt.xlabel("横坐标")
    plt.ylabel("纵坐标")
    # 设置横纵坐标范围
    plt.xticks(range(0, 6))
    plt.yticks(range(0, 16))
    # 设置网格线
    plt.grid(color='black', linestyle='--')
    plt.show()
```

