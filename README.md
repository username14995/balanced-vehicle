# balanced-vehicle
###############################################################################
# 文件名: color_recognition_counter.py
# 功能: K210颜色学习与计数（自动运行版本 - 无需按键）
# 硬件: MaixPy (K210)
# 说明: 
#   1. 上电后自动从ROI区域学习颜色
#   2. 学习完成后自动识别并统计相同颜色的色块数量
#   3. 在LCD上显示颜色名称、当前数量、累计总数和FPS
# 基于源码: color_recognition.py (原作者提供)
# 修改内容: 增加颜色计数、颜色名称识别、累计统计
###############################################################################

import sensor, image, time, lcd

# ========== 硬件初始化 ==========
lcd.init(freq=15000000)
sensor.reset()
sensor.set_pixformat(sensor.RGB565)
sensor.set_framesize(sensor.QVGA)  # 320x240
sensor.skip_frames(time=500)
sensor.set_auto_gain(False)
sensor.set_auto_whitebal(False)
clock = time.clock()

print("Hold the object you want to track in front of the camera in the box.")
print("MAKE SURE THE COLOR OF THE OBJECT YOU WANT TO TRACK IS FULLY ENCLOSED BY THE BOX!")

# ========== ROI区域 (屏幕中央 50x50) ==========
ROI_X = (320 - 50) // 2
ROI_Y = (240 - 50) // 2
ROI_W = 50
ROI_H = 50
r = [(320//2)-(50//2), (240//2)-(50//2), 50, 50]

# ========== 颜色名称映射 ==========
COLOR_NAMES = {
    "Red": [(0, 50, 20, 127, 20, 127)],
    "Green": [(0, 127, -128, -20, -128, 127)],
    "Blue": [(0, 127, -128, 127, -128, -20)],
    "Yellow": [(0, 127, -10, 30, 30, 127)],
    "Purple": [(0, 127, 20, 127, -128, -20)],
    "Orange": [(0, 127, 10, 50, 30, 127)],
    "Cyan": [(0, 127, -128, -10, -10, 30)],
    "Black": [(0, 30, -128, 127, -128, 127)],
    "White": [(100, 127, -128, 127, -128, 127)],
    "Gray": [(40, 80, -128, 127, -128, 127)]
}

# ========== 颜色名称匹配函数 ==========
def get_color_name_from_lab(l_min, l_max, a_min, a_max, b_min, b_max):
    """
    根据LAB阈值判断最接近的颜色名称
    """
    l_center = (l_min + l_max) // 2
    a_center = (a_min + a_max) // 2
    b_center = (b_min + b_max) // 2
    
    best_match = "Unknown"
    best_score = float('inf')
    
    for name, thresholds in COLOR_NAMES.items():
        for th in thresholds:
            score = abs(l_center - (th[0] + th[1]) // 2) + \
                    abs(a_center - (th[2] + th[3]) // 2) + \
                    abs(b_center - (th[4] + th[5]) // 2)
            if score < best_score:
                best_score = score
                best_match = name
    
    return best_match

# ========== 学习颜色阈值 ==========
print("Learning thresholds...")
threshold = [50, 50, 0, 0, 0, 0]  # Middle L, A, B values.

for i in range(50):
    img = sensor.snapshot()
    hist = img.get_histogram(roi=r)
    lo = hist.get_percentile(0.01)  # Get the CDF of the histogram at the 1% range
    hi = hist.get_percentile(0.99)  # Get the CDF of the histogram at the 99% range
    # Average in percentile values.
    threshold[0] = (threshold[0] + lo.l_value()) // 2
    threshold[1] = (threshold[1] + hi.l_value()) // 2
    threshold[2] = (threshold[2] + lo.a_value()) // 2
    threshold[3] = (threshold[3] + hi.a_value()) // 2
    threshold[4] = (threshold[4] + lo.b_value()) // 2
    threshold[5] = (threshold[5] + hi.b_value()) // 2
    for blob in img.find_blobs([threshold], pixels_threshold=100, area_threshold=100, merge=True, margin=10):
        img.draw_rectangle(blob.rect())
        img.draw_cross(blob.cx(), blob.cy())
        img.draw_rectangle(r, color=(0,255,0))
    lcd.display(img)

print("Thresholds learned...")
print("Threshold:", threshold)

# ========== 获取颜色名称 ==========
color_name = get_color_name_from_lab(
    threshold[0], threshold[1],
    threshold[2], threshold[3],
    threshold[4], threshold[5]
)
print("Color Name:", color_name)

# ========== 累计计数器 ==========
total_count = 0
frame_counter = 0

print("Start Color Recognition & Counting...")
print("========================================")

# ========== 主循环 ==========
while(True):
    clock.tick()
    img = sensor.snapshot()
    
    # 检测颜色块
    blobs = img.find_blobs(
        [threshold],
        pixels_threshold=100,
        area_threshold=100,
        merge=True,
        margin=10
    )
    
    # 统计当前帧数量
    current_count = len(blobs)
    
    # 累加总数
    if current_count > 0:
        total_count += current_count
        frame_counter += 1
    
    # 绘制检测结果
    for blob in blobs:
        img.draw_rectangle(blob.rect())
        img.draw_cross(blob.cx(), blob.cy())
        # 在色块上标记编号
        img.draw_string(blob.x(), blob.y() - 15, 
                       "#%d" % (blobs.index(blob) + 1), 
                       color=(255, 255, 0))
    
    # ====== 显示统计信息 ======
    # 颜色名称
    img.draw_string(10, 10, "Color: " + color_name, color=(255, 255, 255))
    # 当前检测数量
    img.draw_string(10, 30, "Count: %d" % current_count, color=(255, 255, 255))
    # 累计总数
    img.draw_string(10, 50, "Total: %d" % total_count, color=(255, 255, 255))
    # FPS
    img.draw_string(10, 70, "FPS: %.1f" % clock.fps(), color=(255, 255, 255))
    
    # 显示ROI区域（淡色标记）
    img.draw_rectangle(r, color=(255, 255, 255), thickness=1)
    
    # 显示到LCD
    lcd.display(img)

# ========== 串口输出功能(可选) ==========
# 如需将数据发送给STM32/51，取消以下注释
"""
from machine import UART

uart = UART(UART.UART1, 115200, 8, None, 1, timeout=1000)

def send_data():
    data = "Color:%s,Count:%d,Total:%d\n" % (color_name, current_count, total_count)
    uart.write(data)

# 在主循环中添加:
# send_data()
"""
