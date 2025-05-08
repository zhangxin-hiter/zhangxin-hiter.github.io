---
title: small-scale fading
date: 2025-05-08
permalink: /posts/2025/05/08/small-scale-fading/
tags: matlab
---

# 瑞利衰落信道
瑞利信道，信号包络幅度服从瑞利分布
matlab提供描述瑞利信道的函数
```
h = comm.RayleighChannel(...
    'SampleRate', fs, ...           %采样频率
    'MaximumDopplerShift', fd, ...  %最大多普勒频移
    'PathDelays', [0 1e-6], ...     %到达接收机的时延
    'AveragePathGains', [0 -3], ... %每条路径的平均功率
    'PathGainsOutputPort', true);   %启用输出各路径的即使信道增益
```
令x为输入信号则
```
[y, pathgain] = h(x)                %y为输出信号，pathgain为信道增益
```
matlab中创建全一矩阵ones函数
```
ones(A);                            %创建A*A的全一矩阵
ones(size(A));                      %创建与A大小一致的全一矩阵
ones(A,B);                          %创建A*B的全一矩阵
```
matlab提供计算fft函数原型
```
Y = fft(X, N);                      %X为输入向量， N为计算长度,Y的范围在0到fs（采样频率之间）
```
matlab提供计算参数N的方法
```
N = x^nextpow2(X);                  %nextpow2计算使得2^p >= n的最小指数p
```
matlab提供生成等间距数组的函数
```
Y = linspace(X1, X2, N);
```

一个瑞利信道带多普勒频移的仿真实例
```
%参数设置
fc = 5e9;                                   %载波频率
v = 120 / 3.6;                              %系统速度
c = 3e8;                                    %光速
fd = (v / c) * fc;                          %最大多普勒频移

%获得采样率
T = 0.015;                                  %采样时间
fs = 100e3;                                 %采样频率100khz
t = (0:1 / fs: T - 1 / fs);                 %获取时间向量

%生成瑞利信道
h = comm.RayleighChannel(... 
    'SampleRate',fs, ...
    'MaximumDopplerShift', fd,...
    'PathGainsOutputPort', true);            %生成瑞利信道
x = ones(length(t), 1);                     %输入信号
[y, PathGain] = h(x);                       %y为输出信号，pathGain为信道增益

%绘制信道增益随时间变化图
figure;
plot(t * 1e3, abs(PathGain));               %单位为毫秒
xlabel('Time(ms)');
ylabel('信道增益|h(t)');
title('瑞利信道增益随时间变化');
grid on;

%绘制多普勒谱
NFFT = 2^nextpow2(length(y));
Y = fftshift(fft(y, NFFT));
f_axis = linspace(-fs / 2, fs / 2, NFFT);
P = abs(Y).^2 / (NFFT * fs);

%绘制多普勒谱
figure;
plot(f_axis, 10 * log10(P));
xlabel('频率(hz)');
ylabel('功率/频率');
title('多普勒谱');
grid on;
xlim([-2*fd, 2*fd]);  % 限制显示主要能量区域
```

