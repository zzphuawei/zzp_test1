下面是一个Python函数，用于根据给定的LED灯数据格式 `[(时间戳, 颜色)]` 识别是否包含白灯常亮和白灯闪烁。函数考虑了颜色变化的时间段，并通过分析连续白色段的持续时间和交替模式来区分常亮和闪烁。

```python
def detect_white_states(data, constant_threshold=1.0):
    """
    识别LED数据中是否包含白灯常亮和白灯闪烁。

    参数:
        data: list of (timestamp, color)
              每个元素表示在timestamp时刻颜色变为color，
              颜色为字符串，如"black"、"white"等。
        constant_threshold: float, 常亮的最小持续时间（与时间戳单位一致），
                            默认为1.0。若白色段持续时间大于此值则可能为常亮。

    返回:
        (has_constant, has_blinking): 两个布尔值，分别表示是否存在常亮和闪烁。
    """
    if len(data) < 2:
        return False, False

    # 按时间戳排序（确保有序）
    data_sorted = sorted(data, key=lambda x: x[0])

    # 构建时间段列表，每个时间段有颜色和持续时间
    segments = []  # 每个元素为 (color, duration)
    for i in range(len(data_sorted) - 1):
        t1, c1 = data_sorted[i]
        t2, _ = data_sorted[i + 1]
        duration = t2 - t1
        segments.append((c1, duration))
    # 最后一个点没有结束时间，忽略（根据题目描述最后应是不亮，故无影响）

    # 将连续只包含黑白颜色的时间段分组为块（相邻颜色不同，块内自然交替）
    blocks = []
    current_block = []
    for seg in segments:
        color, _ = seg
        if color in ("black", "white"):
            current_block.append(seg)
        else:
            if current_block:
                blocks.append(current_block)
                current_block = []
    if current_block:
        blocks.append(current_block)

    has_constant = False
    has_blinking = False

    # 处理每个块
    for block in blocks:
        # 收集块中所有白色段及其持续时间
        white_segments = [(i, dur) for i, (c, dur) in enumerate(block) if c == "white"]
        white_count = len(white_segments)

        if white_count >= 2:
            # 存在至少两个白色段，则一定有闪烁（因为块内交替）
            has_blinking = True

            # 检查是否有某个白色段持续时间明显长于其他（可能为常亮）
            # 取最长和第二长的持续时间进行比较
            durations = [dur for _, dur in white_segments]
            durations.sort(reverse=True)
            longest = durations[0]
            second_longest = durations[1] if len(durations) > 1 else 0
            # 若最长远大于第二长（例如2倍），且超过阈值，则认为是常亮
            if longest > constant_threshold and longest > 2 * second_longest:
                has_constant = True
        elif white_count == 1:
            # 只有一个白色段，若持续时间超过阈值则为常亮
            dur = white_segments[0][1]
            if dur > constant_threshold:
                has_constant = True

    return has_constant, has_blinking


# 示例用法
if __name__ == "__main__":
    # 构造符合描述的数据：先不亮，常亮白色，不亮，白色闪烁，不亮
    data = [
        (0, "black"),      # 开始不亮
        (1, "white"),      # 常亮开始
        (3, "black"),      # 常亮结束，不亮
        (3.1, "white"),    # 闪烁第一次亮
        (3.2, "black"),    # 灭
        (3.3, "white"),    # 第二次亮
        (3.4, "black"),    # 灭
        (3.5, "white"),    # 第三次亮
        (3.6, "black")     # 闪烁结束，最后不亮
    ]

    constant, blinking = detect_white_states(data, constant_threshold=0.5)
    print(f"包含白灯常亮: {constant}")   # 应为 True
    print(f"包含白灯闪烁: {blinking}")   # 应为 True
```

### 说明
1. **时间段构建**：将连续的时间变化点转换为时间段，每个时间段有明确的颜色和持续时间（从当前时间戳到下一个时间戳）。
2. **颜色块划分**：将只包含黑白色的连续时间段分为一个块，块内颜色自然交替（因为原始数据相邻时间点颜色必不同）。
3. **闪烁检测**：若一个块内白色段数量 ≥ 2，则存在闪烁（因为交替出现）。
4. **常亮检测**：
   - 若块内只有一个白色段且持续时间超过阈值，则为常亮。
   - 若块内有多个白色段，则取最长与第二长比较，若最长远大于第二长且超过阈值，则最长段为常亮（其余为闪烁）。
5. **阈值**：`constant_threshold` 用于区分常亮与短暂亮起，可根据实际时间单位调整（如秒）。示例中设为0.5秒。