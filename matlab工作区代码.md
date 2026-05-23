```工作区参数加载
%% 1. 接触网参数
alpha1 = 0.4665;
alpha2 = 0.0832;
alpha3 = 0.2603;
alpha4 = -0.2801;
alpha5 = -0.3364;
k0 = 3684.5;    % 接触网平均刚度 N/m
L = 50;         % 接触网跨距 m
L1 = 8;         % 相邻吊弦间距 m

%% 2. 受电弓参数
m1 = 7.12;      % 弓头等效质量 kg
m2 = 6;       % 上框架等效质量 kg
m3 = 5.8;        % 下框架等效质量 kg
k1 = 9430;      % 弓头等效刚度 N/m
k2 = 14100;     % 上框架等效刚度 N/m
k3 = 0.1;         % 下框架等效刚度 N/m
c1 = 0;        % 弓头等效阻尼 N·s/m
c2 = 0;        % 上框架等效阻尼 N·s/m
c3 = 70;       % 下框架等效阻尼 N·s/m

%% 3. 列车运行与弓网力参数
v_kmh = 250;                % 列车运行速度 km/h
v = v_kmh / 3.6;        % 转为m/s
F0 = 70;                    % 静态抬升力 N
k_aero = 0.00097;          % 空气动力系数
Fa = k_aero * v_kmh^2;     % 气动升力 N
Fd = F0;                    % 底座静态抬升力 N
u = 0;                      % 主动控制力，被动仿真设为0，后续加控制可修改
noise_coef = 0.01;         % 白噪声扰动系数
```



```读取Simpack文件
% 读取 ASCII 文件
data1 = readmatrix('.txt'); 
body_vel.time = data1(:,1);
body_vel.signals.values = data1(:,2);
body_vel.signals.dimensions = 1;
% 读取 ASCII 文件
data2 = readmatrix('.txt'); 
body_pos.time = data2(:,1);
body_pos.signals.values = data2(:,2);
body_pos.signals.dimensions = 1;
```



```特征值导出
% 读取仿真导出的数据
F = out.contact_force;  % 弓网接触力数组
X = out.x1;             % 弓头位移数组

% 1. 接触力平均值
Fm = mean(F);
% 2. 接触力标准差
s = std(F);
% 3. 接触力最大值、最小值
Fmax = max(F);
Fmin = min(F);
% 4. 弓头位移标准差
S_wy = std(X);
% 5. 弓头位移振幅
a = (max(X) - min(X))/2;
% 6. 目标接触力Fb
Fb = F0 + 0.00097*v_kmh^2;
% 7. 平均值合规性
ratio = Fm/Fb;

% 输出结果
fprintf('=====弓网耦合仿真评价指标=====\n');
fprintf('目标接触力Fb：%.2f N\n',Fb);
fprintf('接触力平均值Fm：%.2f N，合规性：%.2f %%\n',Fm,ratio*100);
fprintf('接触力标准差s：%.2f N\n',s);
fprintf('接触力最大值：%.2f N，最小值：%.2f N\n',Fmax,Fmin);
fprintf('弓头位移标准差：%.6f m\n',S_wy);
fprintf('弓头位移振幅：%.4f m\n',a);

% 读取仿真导出的数据
F = out.contact_force;  % 弓网接触力数组
X = out.x1;             % 弓头位移数组

% 1. 接触力平均值
Fm = mean(F);
% 2. 接触力标准差
s = std(F);
% 3. 接触力最大值、最小值
Fmax = max(F);
Fmin = min(F);

% 输出结果
fprintf('接触力平均值Fm：%.2f N',Fm);
fprintf('接触力标准差s：%.2f N\n',s);
fprintf('接触力最大值：%.2f N，最小值：%.2f N\n',Fmax,Fmin);
```



```示意图导出
%%1. 数据提取与预处理

t = out.tout;
F = out.contact_force;    % 弓网接触力
x1 = out.x1;              % 弓头位移
k_t = out.k_t;            % 接触网刚度

%% 统一长度+维度
min_len = min([length(t), length(F), length(x1), length(k_t)]);
% 截断所有变量到相同长度
t = t(1:min_len, :);
F = F(1:min_len, :);
x1 = x1(1:min_len, :);
k_t = k_t(1:min_len, :);
% 强制转成列向量，避免维度不匹配
t = t(:);
F = F(:);
x1 = x1(:);
k_t = k_t(:);

%% 截取0s-5s的波形
time_range = t >= 0 & t <= 10; % 仅保留0到10数据
t_stable = t(time_range);
F_stable = F(time_range);
x1_stable = x1(time_range);
k_t_stable = k_t(time_range);

%% 图1：独立的接触网刚度图
figure('Position', [100, 100, 1000, 500], 'Name', '接触网刚度变化');
plot(t_stable, k_t_stable, 'k-', 'LineWidth', 1.2);
hold on;
yline(mean(k_t_stable), 'k--', 'LineWidth', 1.5, 'Label', '刚度平均值');
xlabel('时间 (s)');
ylabel('接触网刚度 (N/m)');
title('接触网刚度随时间变化曲线');
legend('接触网刚度', '接触网刚度平均值', 'Location', 'best');
grid on;
xlim([0, 10]); % 强制横轴显示0-10s
% 可选：保存图片，取消注释即可
% saveas(gcf, '接触网刚度图.png');

%% 图2：独立的弓头位移图
figure('Position', [100, 100, 1000, 500], 'Name', '弓头位移变化');
plot(t_stable, x1_stable*1000, 'k-', 'LineWidth', 1.2); % 转成毫米，更直观
hold on;
yline(mean(x1_stable*1000), 'k--', 'LineWidth', 1.5, 'Label', '弓头位移平均值');
xlabel('时间 (s)');
ylabel('弓头位移 (mm)');
title('弓头垂向位移随时间变化曲线');
legend('弓头位移', '弓头位移平均值', 'Location', 'best');
grid on;
xlim([0, 10]); % 强制横轴显示0-10s
% 可选：保存图片，取消注释即可
% saveas(gcf, '弓头位移图.png');

%% 图3：独立的弓网接触力图
figure('Position', [100, 100, 1000, 500], 'Name', '弓网接触力变化');
plot(t_stable, F_stable, 'k-', 'LineWidth', 1.2);
hold on;
yline(mean(F_stable), 'k--', 'LineWidth', 1.5, 'Label', '接触力平均值');
yline(20, 'k:', 'LineWidth', 1.5, 'Label', '20N安全下限');
xlabel('时间 (s)');
ylabel('弓网接触力 (N)');
title('弓网接触力随时间变化曲线');
legend('弓网接触力', '接触力平均值', '20N安全下限', 'Location', 'best');
grid on;
xlim([0, 10]); % 强制横轴显示0-10s
% 可选：保存图片，取消注释即可
% saveas(gcf, '弓网接触力图');
```



```xls文件导出
% 1. 对齐维度
time = out.tout(1:10001, :);  
force = out.contact_force;
displacement = out.x1 * 1000;  
stiffness = out.k_t;
s = out.dis.Data(1:10001, :);

% 2. 构建表格
data = table(time, force, displacement, stiffness, s,...
    'VariableNames', {'时间(s)', '接触力(N)', '弓头位移(mm)', '接触网刚度(N/m)', '运行距离(m)'});

% 3. 导出Excel
writetable(data, '200.xlsx');
disp('✅ 数据导出成功！');
```

```
% 1. 读取仿真导出的时间向量和数据
t = out.tout;           % 获取仿真时间数组 (注意：如果你的时间变量名叫别的，请替换)
F_all = out.contact_force;  % 全局弓网接触力数组
X_all = out.x1;             % 全局弓头位移数组

% 2. 确定 3~10s 的时间索引
% find 找出满足条件的索引位置
idx = find(t >= 3 & t <= 9.9940); 

% 3. 截取 3~10s 内的数据片段
F = F_all(idx);
X = X_all(idx);

% ========== 下面的计算代码保持不变，但此时的 F 和 X 已经是 3~10s 的数据了 ==========

% 1. 接触力平均值
Fm = mean(F);
% 2. 接触力标准差
s = std(F);
% 3. 接触力最大值、最小值
Fmax = max(F);
Fmin = min(F);
% 4. 弓头位移标准差
S_wy = std(X);
% 5. 弓头位移振幅
a = (max(X) - min(X))/2;

% (假设 F0 和 v_kmh 在前面已经定义)
% 6. 目标接触力Fb
Fb = F0 + 0.00097*v_kmh^2;
% 7. 平均值合规性
ratio = Fm/Fb;

% 输出结果
fprintf('===== 3-10s 弓网耦合仿真评价指标 =====\n');
fprintf('目标接触力Fb：%.2f N\n',Fb);
fprintf('接触力平均值Fm：%.2f N，合规性：%.2f %%\n',Fm,ratio*100);
fprintf('接触力标准差s：%.2f N\n',s);
fprintf('接触力最大值：%.2f N，最小值：%.2f N\n',Fmax,Fmin);
fprintf('弓头位移标准差：%.6f m\n',S_wy);
fprintf('弓头位移振幅：%.4f m\n',a);
```

