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
% 读取垂向速度
data1 = readmatrix('.txt'); 
body_vel.time = data1(:,1);
body_vel.signals.values = data1(:,2);
body_vel.signals.dimensions = 1;
% 读取垂向位移
data2 = readmatrix('.txt'); 
body_pos.time = data2(:,1);
body_pos.signals.values = data2(:,2);
body_pos.signals.dimensions = 1;
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
disp('数据导出成功！');
```
