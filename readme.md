# 项目使用说明  2025.6.18更新日志
删除了部分冗余文件

## 更新使用yaml来保存与修改重要参数 需要额外安装pyhton的yaml库

打开 开始菜单 → Anaconda3 (64-bit) → Anaconda Prompt (Anaconda3)

输入 activate #环境名字#    (回车)    激活虚拟环境
输入 pip install yaml      (回车)    给环境安装yaml库


# 项目结构
config.yaml: 配置文件，用于设置实验参数。
AI_Tip_TPM_Main_fixed_V3.py: 主程序，运行实验。


# 修改配置文件
打开项目目录下的 config.yaml 文件，根据实验需求修改参数。
修改完成后，记得保存yaml文件。




# 2025.7.25更新日志
config.yaml增加了如下参数：
max_rescan_times
bias_rise_factor
current_rise_factor
key_point_ditection_factor

现在可以对同一区域重复扫描并检查是否完全反应了。
max_rescan_times用来控制最大重扫次数。
多次次扫描同一区域时会根据bias_rise_factor与current_rise_factor来提高针尖操作参数。

可以通过key_point_ditection_factor来控制key_point算法的检测范围。

更新了key point算法的权重参数   best_0723.pt
请将best_0723.pt放置到key_point文件夹内。

# 项目结构
config.yaml: 配置文件，用于设置实验参数。
AI_Tip_TPM_Main_fixed_V3.2.py: 主程序，运行实验。



# 2025.7.30更新日志
修改了扫描框不会移动的bug

# 项目结构
config.yaml: 配置文件，用于设置实验参数。
AI_Tip_TPM_Main_fixed_V3.3.py: 主程序，运行实验。

# 2025.8.18更新日志
现在支持连续打出大面积的honeycomb和kagome
在config.yaml中调整参数  lattice_shape 为'honeycomb_lattice_all'或者'kagome_lattice_all'即可
建议在执行代码前，将扫描框之间的移动步长降低，保证相邻的扫描框之间至少有大约1/3~1/2的重叠。

# 项目结构
config.yaml: 配置文件，用于设置实验参数。
AI_Tip_TPM_Main_fixed_V3.4.py: 主程序，运行实验。