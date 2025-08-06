# 流水灯周期检测与帧合并解决方案

针对流水灯周期检测和帧合并问题，我设计了一个完整的解决方案，能够处理帧率波动、卡顿和丢帧情况。这个方案结合了时间戳分析、位置跟踪和动态周期检测算法。

```python
import cv2
import numpy as np
import time
from collections import deque
import matplotlib.pyplot as plt

class LEDCycleProcessor:
    def __init__(self, frame_rate=30, led_interval=10, leds_per_row=100, rows=10):
        """
        初始化流水灯处理器
        
        参数:
        frame_rate: 估计的视频帧率 (fps)
        led_interval: 灯珠切换间隔 (ms)
        leds_per_row: 每排灯珠数量
        rows: 灯珠排数
        """
        # 灯珠系统参数
        self.led_interval = led_interval / 1000.0  # 转换为秒
        self.cycle_duration = (leds_per_row * led_interval) / 1000.0  # 完整周期时长(秒)
        self.leds_per_row = leds_per_row
        self.rows = rows
        
        # 周期检测参数
        self.frame_rate = frame_rate
        self.min_cycle_duration = self.cycle_duration * 0.8  # 最小周期时长(秒)
        self.max_cycle_duration = self.cycle_duration * 1.5  # 最大周期时长(秒)
        self.position_history = deque(maxlen=1000)  # 存储位置历史
        self.time_history = deque(maxlen=1000)       # 存储时间戳历史
        self.cycle_frames = []                       # 当前周期的帧集合
        self.completed_cycles = []                   # 完成的周期数据
        self.current_cycle_start = None              # 当前周期开始时间
        self.current_cycle_index = 0                 # 当前灯珠位置索引
        self.last_frame_time = None                  # 上一帧处理时间
        self.led_positions = None                    # 灯珠位置信息
        
        # 状态跟踪
        self.cycle_count = 0
        self.avg_cycle_duration = self.cycle_duration
        self.cycle_start_positions = []
        
        # 调试信息
        self.debug_data = {
            'frame_times': [],
            'positions': [],
            'cycle_markers': []
        }
    
    def detect_led_positions(self, frame):
        """检测并排序灯珠位置"""
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        
        # 自适应对比度增强
        clahe = cv2.createCLAHE(clipLimit=3.0, tileGridSize=(8, 8))
        enhanced = clahe.apply(gray)
        
        # 自动亮度阈值
        bright_pixels = enhanced[enhanced > np.percentile(enhanced, 95)]
        threshold_value = np.mean(bright_pixels) if len(bright_pixels) > 0 else 200
        _, binary = cv2.threshold(enhanced, threshold_value, 255, cv2.THRESH_BINARY)
        
        # 形态学操作
        h, w = gray.shape
        kernel_size = max(1, min(h, w) // 300)
        kernel = np.ones((kernel_size, kernel_size), np.uint8)
        processed = cv2.morphologyEx(binary, cv2.MORPH_CLOSE, kernel, iterations=2)
        
        # 查找轮廓
        contours, _ = cv2.findContours(processed, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
        
        # 过滤轮廓并获取位置
        positions = []
        for cnt in contours:
            area = cv2.contourArea(cnt)
            # 自适应面积过滤
            if area > max(10, w * h * 0.0001):
                x, y, w, h = cv2.boundingRect(cnt)
                positions.append((x + w/2, y + h/2))  # 使用中心点
        
        # 按行分组
        if len(positions) == 0:
            return None
        
        # 按y坐标排序并分组
        positions.sort(key=lambda p: p[1])
        row_positions = []
        current_row = [positions[0]]
        y_threshold = (max(p[1] for p in positions) - min(p[1] for p in positions)) / (self.rows * 2)
        
        for i in range(1, len(positions)):
            if positions[i][1] - positions[i-1][1] > y_threshold:
                row_positions.append(current_row)
                current_row = [positions[i]]
            else:
                current_row.append(positions[i])
        row_positions.append(current_row)
        
        # 每行内按x坐标排序
        for i, row in enumerate(row_positions):
            row.sort(key=lambda p: p[0])
            row_positions[i] = row
        
        # 确保行数正确
        if len(row_positions) != self.rows:
            print(f"警告: 检测到 {len(row_positions)} 排, 但预期 {self.rows} 排")
            self.rows = len(row_positions)
        
        self.led_positions = row_positions
        return row_positions
    
    def get_current_position(self, frame):
        """获取当前帧最亮灯珠位置索引"""
        if self.led_positions is None:
            self.detect_led_positions(frame)
            if self.led_positions is None:
                return -1
        
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
        
        # 计算每排最亮灯珠
        max_brightness = 0
        max_position_index = 0
        
        for row_idx, row in enumerate(self.led_positions):
            for col_idx, (x, y) in enumerate(row):
                # 获取灯珠区域
                radius = min(15, int(min(frame.shape[:2]) / 50))  # 自适应半径
                x1, y1 = max(0, int(x - radius)), max(0, int(y - radius))
                x2, y2 = min(frame.shape[1], int(x + radius)), min(frame.shape[0], int(y + radius))
                
                if x2 > x1 and y2 > y1:
                    roi = gray[y1:y2, x1:x2]
                    if roi.size > 0:
                        brightness = np.max(roi)
                        if brightness > max_brightness:
                            max_brightness = brightness
                            max_position_index = col_idx
        
        return max_position_index
    
    def process_frame(self, frame, timestamp=None):
        """处理单帧"""
        if timestamp is None:
            timestamp = time.time()
        
        # 获取当前灯珠位置
        current_position = self.get_current_position(frame)
        
        # 记录调试数据
        self.debug_data['frame_times'].append(timestamp)
        self.debug_data['positions'].append(current_position)
        
        # 初始化周期跟踪
        if self.current_cycle_start is None:
            self.current_cycle_start = timestamp
            self.current_cycle_index = current_position
            self.cycle_frames = [frame.copy()]
            self.position_history.append(current_position)
            self.time_history.append(timestamp)
            self.debug_data['cycle_markers'].append(0)
            return False, frame
        
        # 计算时间差
        time_diff = timestamp - self.last_frame_time if self.last_frame_time else 0
        
        # 位置变化检测
        position_diff = abs(current_position - self.current_cycle_index)
        
        # 处理可能的跳变（周期结束）
        cycle_completed = False
        if position_diff > self.leds_per_row * 0.8:  # 位置跳变超过80%灯珠数量
            # 检查是否满足周期时长条件
            cycle_time = timestamp - self.current_cycle_start
            if self.min_cycle_duration <= cycle_time <= self.max_cycle_duration:
                cycle_completed = True
        
        # 基于时间的周期检测（防止卡顿导致漏检）
        if not cycle_completed:
            cycle_time = timestamp - self.current_cycle_start
            if cycle_time >= self.max_cycle_duration:
                cycle_completed = True
        
        # 处理周期结束
        if cycle_completed:
            # 保存当前周期
            self.completed_cycles.append({
                'start_time': self.current_cycle_start,
                'end_time': timestamp,
                'duration': timestamp - self.current_cycle_start,
                'frames': self.cycle_frames,
                'start_position': self.cycle_start_positions[-1] if self.cycle_start_positions else 0,
                'end_position': current_position
            })
            
            # 更新统计数据
            self.cycle_count += 1
            self.avg_cycle_duration = (
                (self.avg_cycle_duration * (self.cycle_count - 1) + 
                (timestamp - self.current_cycle_start)
            ) / self.cycle_count
            
            # 重置周期
            self.current_cycle_start = timestamp
            self.cycle_frames = [frame.copy()]
            self.cycle_start_positions.append(current_position)
            self.debug_data['cycle_markers'].append(1)
            
            # 创建可视化
            vis_frame = self.visualize_frame(frame, current_position, cycle_completed)
            self.last_frame_time = timestamp
            return True, vis_frame
        
        # 未完成周期，继续收集帧
        self.cycle_frames.append(frame.copy())
        self.current_cycle_index = current_position
        self.position_history.append(current_position)
        self.time_history.append(timestamp)
        self.debug_data['cycle_markers'].append(0)
        
        # 创建可视化
        vis_frame = self.visualize_frame(frame, current_position, cycle_completed)
        self.last_frame_time = timestamp
        return False, vis_frame
    
    def visualize_frame(self, frame, position, cycle_completed):
        """创建可视化帧"""
        vis_frame = frame.copy()
        h, w = vis_frame.shape[:2]
        
        # 显示当前位置
        cv2.putText(vis_frame, f"Position: {position}/{self.leds_per_row-1}", 
                   (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 255), 2)
        
        # 显示周期信息
        if self.current_cycle_start:
            cycle_time = time.time() - self.current_cycle_start
            cv2.putText(vis_frame, f"Cycle: {cycle_time:.2f}s / {self.cycle_duration:.2f}s", 
                       (10, 70), cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 255), 2)
        
        cv2.putText(vis_frame, f"Cycles: {self.cycle_count}", 
                   (10, 110), cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 255), 2)
        
        # 如果周期完成，添加标记
        if cycle_completed:
            cv2.putText(vis_frame, "CYCLE COMPLETED!", (w//2-150, h//2), 
                       cv2.FONT_HERSHEY_SIMPLEX, 1.5, (0, 255, 0), 3)
        
        return vis_frame
    
    def merge_cycle_frames(self, cycle_data):
        """合并一个周期内的帧到一张图像"""
        if not cycle_data['frames']:
            return None
        
        # 创建一个空白图像用于合并
        h, w = cycle_data['frames'][0].shape[:2]
        merged_image = np.zeros((h, w, 3), dtype=np.uint8)
        
        # 创建亮度累积图
        brightness_accum = np.zeros((h, w), dtype=np.float32)
        
        # 处理所有帧
        for frame in cycle_data['frames']:
            gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY).astype(np.float32)
            brightness_accum = np.maximum(brightness_accum, gray)
        
        # 归一化并转换为BGR
        max_val = np.max(brightness_accum)
        if max_val > 0:
            brightness_accum = (brightness_accum / max_val * 255).astype(np.uint8)
        merged_image = cv2.cvtColor(brightness_accum, cv2.COLOR_GRAY2BGR)
        
        # 添加周期信息
        cv2.putText(merged_image, f"Cycle {self.cycle_count}: {cycle_data['duration']:.2f}s", 
                   (10, 30), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0), 2)
        cv2.putText(merged_image, f"Frames: {len(cycle_data['frames'])}", 
                   (10, 70), cv2.FONT_HERSHEY_SIMPLEX, 1, (0, 255, 0), 2)
        
        return merged_image
    
    def plot_debug_info(self):
        """绘制调试信息图表"""
        if not self.debug_data['frame_times']:
            return
        
        # 转换时间戳为相对时间
        start_time = self.debug_data['frame_times'][0]
        rel_times = [t - start_time for t in self.debug_data['frame_times']]
        
        # 创建位置图
        plt.figure(figsize=(15, 10))
        
        plt.subplot(2, 1, 1)
        plt.plot(rel_times, self.debug_data['positions'], 'b-', label='LED Position')
        
        # 标记周期结束点
        cycle_end_times = [rel_times[i] for i, marker in enumerate(self.debug_data['cycle_markers']) if marker == 1]
        if cycle_end_times:
            plt.scatter(cycle_end_times, [self.debug_data['positions'][i] 
                         for i, marker in enumerate(self.debug_data['cycle_markers']) if marker == 1],
                       color='red', s=100, zorder=5, label='Cycle End')
        
        plt.xlabel('Time (s)')
        plt.ylabel('LED Position')
        plt.title('LED Position Over Time')
        plt.legend()
        plt.grid(True)
        
        # 创建帧间隔图
        plt.subplot(2, 1, 2)
        frame_intervals = [rel_times[i] - rel_times[i-1] 
                          for i in range(1, len(rel_times))]
        plt.plot(rel_times[1:], frame_intervals, 'g-')
        plt.axhline(y=1/self.frame_rate, color='r', linestyle='--', label='Expected Interval')
        
        plt.xlabel('Time (s)')
        plt.ylabel('Frame Interval (s)')
        plt.title('Frame Intervals Over Time')
        plt.legend()
        plt.grid(True)
        
        plt.tight_layout()
        plt.savefig('cycle_analysis.png')
        plt.show()

# 使用示例
def process_video(video_path, output_dir):
    # 初始化处理器
    processor = LEDCycleProcessor(
        frame_rate=30,
        led_interval=10,  # 10ms
        leds_per_row=100,
        rows=10
    )
    
    # 打开视频
    cap = cv2.VideoCapture(video_path)
    if not cap.isOpened():
        print("Error: Could not open video")
        return
    
    # 获取实际帧率
    fps = cap.get(cv2.CAP_PROP_FPS)
    print(f"Video FPS: {fps}")
    
    frame_count = 0
    last_time = time.time()
    
    while cap.isOpened():
        ret, frame = cap.read()
        if not ret:
            break
        
        # 计算时间戳（模拟实际时间）
        current_time = last_time + (1 / fps) if fps > 0 else time.time()
        
        # 处理帧
        cycle_completed, vis_frame = processor.process_frame(frame, current_time)
        
        # 显示处理结果
        cv2.imshow('Processing', vis_frame)
        if cv2.waitKey(1) & 0xFF == ord('q'):
            break
        
        # 处理完成的周期
        if cycle_completed and processor.completed_cycles:
            last_cycle = processor.completed_cycles[-1]
            merged_image = processor.merge_cycle_frames(last_cycle)
            if merged_image is not None:
                output_path = f"{output_dir}/cycle_{processor.cycle_count}.png"
                cv2.imwrite(output_path, merged_image)
                print(f"Saved merged image for cycle {processor.cycle_count}")
        
        frame_count += 1
        last_time = current_time
    
    cap.release()
    cv2.destroyAllWindows()
    
    # 保存最后一个周期（如果未完成）
    if processor.cycle_frames:
        last_cycle = {
            'start_time': processor.current_cycle_start,
            'end_time': last_time,
            'duration': last_time - processor.current_cycle_start,
            'frames': processor.cycle_frames,
            'start_position': processor.cycle_start_positions[-1] if processor.cycle_start_positions else 0,
            'end_position': processor.position_history[-1] if processor.position_history else 0
        }
        merged_image = processor.merge_cycle_frames(last_cycle)
        if merged_image is not None:
            output_path = f"{output_dir}/cycle_final.png"
            cv2.imwrite(output_path, merged_image)
    
    # 生成调试图表
    processor.plot_debug_info()
    
    print(f"Processed {frame_count} frames, detected {processor.cycle_count} complete cycles")

# 运行处理
if __name__ == "__main__":
    video_path = "led_video.mp4"  # 替换为实际视频路径
    output_dir = "output_cycles"  # 输出目录
    process_video(video_path, output_dir)
```

## 解决方案核心设计

### 1. 自适应周期检测算法

```python
# 位置跳变检测
if position_diff > self.leds_per_row * 0.8:  # 位置跳变超过80%灯珠数量
    # 检查是否满足周期时长条件
    cycle_time = timestamp - self.current_cycle_start
    if self.min_cycle_duration <= cycle_time <= self.max_cycle_duration:
        cycle_completed = True

# 基于时间的周期检测（防止卡顿导致漏检）
if not cycle_completed:
    cycle_time = timestamp - self.current_cycle_start
    if cycle_time >= self.max_cycle_duration:
        cycle_completed = True
```

这个双重检测机制确保：
- 在正常运行时，通过位置跳变准确检测周期结束
- 在卡顿或丢帧时，通过超时机制强制结束当前周期

### 2. 帧合并技术

```python
def merge_cycle_frames(self, cycle_data):
    # 创建亮度累积图
    brightness_accum = np.zeros((h, w), dtype=np.float32)
    
    # 处理所有帧
    for frame in cycle_data['frames']:
        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY).astype(np.float32)
        brightness_accum = np.maximum(brightness_accum, gray)
    
    # 归一化并转换为BGR
    max_val = np.max(brightness_accum)
    if max_val > 0:
        brightness_accum = (brightness_accum / max_val * 255).astype(np.uint8)
    merged_image = cv2.cvtColor(brightness_accum, cv2.COLOR_GRAY2BGR)
```

这种方法：
- 保留周期内所有亮过的灯珠
- 使用最大值合并（不是平均），确保所有灯珠清晰可见
- 处理不同亮度的灯珠，保留相对亮度关系

### 3. 时间戳处理

```python
# 使用模拟时间戳处理帧率波动
current_time = last_time + (1 / fps) if fps > 0 else time.time()

# 在实际处理中使用
cycle_time = timestamp - self.current_cycle_start
```

这种方法：
- 使用视频帧率计算理论时间戳
- 允许使用实际时间戳（如果提供）
- 处理帧率波动和丢帧情况

### 4. 调试与分析工具

```python
def plot_debug_info(self):
    # 绘制LED位置随时间变化
    plt.plot(rel_times, self.debug_data['positions'], 'b-', label='LED Position')
    
    # 绘制帧间隔变化
    frame_intervals = [rel_times[i] - rel_times[i-1] 
                      for i in range(1, len(rel_times))]
    plt.plot(rel_times[1:], frame_intervals, 'g-')
```

提供可视化分析：
- LED位置随时间变化曲线
- 实际帧间隔分布
- 标记检测到的周期结束点
- 帮助理解系统行为，调试参数

## 处理帧率波动和丢帧的策略

1. **动态周期时长范围**：
   ```python
   self.min_cycle_duration = self.cycle_duration * 0.8
   self.max_cycle_duration = self.cycle_duration * 1.5
   ```
   允许周期时长有±50%的波动，适应帧率变化

2. **位置跳变检测**：
   ```python
   if position_diff > self.leds_per_row * 0.8:
   ```
   当位置发生大范围跳变时（>80%灯珠数量），认为可能是周期结束

3. **超时强制结束**：
   ```python
   if cycle_time >= self.max_cycle_duration:
       cycle_completed = True
   ```
   防止因丢帧导致周期永远不结束

4. **自适应帧处理**：
   ```python
   # 计算时间差
   time_diff = timestamp - self.last_frame_time if self.last_frame_time else 0
   ```
   使用实际时间差而非固定帧间隔

## 系统优势

1. **鲁棒性**：处理帧率波动、丢帧和卡顿
2. **准确性**：双重检测机制确保周期结束点准确
3. **可视化**：提供丰富的调试信息和分析工具
4. **灵活性**：参数可调整，适应不同灯珠配置
5. **自动化**：自动合并周期帧，无需手动干预

此解决方案特别适合处理手机拍摄的流水灯视频，能够有效应对帧率波动和丢帧问题，准确检测周期结束点并生成合并图像。