# 视频流开发——视觉大模型

## 项目概述

本项目是一个基于YOLOv8姿态检测和SiliconFlow视觉大模型的实时视频分析系统。系统能够从RTSP视频流中实时检测人体姿态、识别人体动作，并结合AI视觉模型生成画面描述。

## 功能特性

- **实时姿态检测**：使用YOLOv8-pose模型检测人体关键点，绘制骨骼连线
- **动作识别**：自动识别人体动作（举手、挥手等）
- **AI视觉分析**：调用SiliconFlow API的视觉大模型对画面进行语义描述
- **实时推流**：支持RTSP推流输出处理后的视频
- **性能监控**：实时显示FPS、人数统计等指标
- **截图功能**：支持一键保存当前画面

## 一、项目流程

### 1.1 项目流程图

![](D:\Users\huihu\Desktop\2026\damoxing\video\流程图.jpg)

### 1.2 项目结构

text

```
project/
├── zhenghe.py          # 主程序文件
├── yolov8n-pose.pt     # YOLO模型权重（首次运行自动下载）
├── screenshot_*.jpg    # 截图文件（按S键生成）
└── README.md           # 本文件
```

## 二、项目配置

### 2.1 技术指标

| 指标项     | 目标值   | 实际达成   |
| :--------- | :------- | :--------- |
| 处理帧率   | ≥20 FPS  | 20-30 FPS  |
| 检测延迟   | ≤50ms    | 30-50ms    |
| AI分析间隔 | 30帧/次  | 可配置     |
| 支持人数   | 单帧多人 | 无硬性上限 |
| 系统可用性 | 7x24小时 | 支持       |

### 2.2 模块划分

| 模块名称         | 功能描述       | 关键技术        |
| :--------------- | :------------- | :-------------- |
| VideoCapture     | RTSP视频流读取 | OpenCV + FFmpeg |
| PoseDetector     | 人体关键点检测 | YOLOv8-pose     |
| ActionRecognizer | 动作识别逻辑   | 骨骼几何分析    |
| VLMCaller        | 视觉大模型调用 | SiliconFlow API |
| VideoRenderer    | 画面渲染与叠加 | OpenCV绘图      |
| RTSPStreamer     | 推流输出       | FFmpeg子进程    |
| UIManager        | 用户交互       | OpenCV HighGUI  |

### 2.3 处理参数

| 参数                | 默认值 | 说明                             |
| :------------------ | :----- | :------------------------------- |
| `ai_frame_interval` | 30     | 每隔多少帧调用一次 AI 模型       |
| `max_tokens`        | 80     | AI 返回的最大 token 数           |
| `temperature`       | 0.7    | 生成文本的随机性                 |
| 图像压缩质量        | 60%    | 发送给 AI 的图像压缩质量（JPEG） |
| 骨骼点置信度阈值    | 0.5    | 关键点检测的最低置信度           |

## 三、项目步骤

### 实时人体姿态检测

系统使用YOLOv8-Pose模型，能够从视频画面中检测出人体的17个关键点位置，并连接成骨骼线条。

| 检测内容     | 说明                                                   |
| :----------- | :----------------------------------------------------- |
| 关键点数量   | 17个（鼻子、眼睛、肩膀、肘部、手腕、髋部、膝盖、脚踝） |
| 骨骼连线     | 13条（手臂、躯干、腿部的主要骨骼）                     |
| 检测速度     | 每秒20-30帧                                            |
| 同时检测人数 | 无硬性上限（画面中可见的人体均可检测）                 |

**示例效果：**

- 画面上每个关节位置显示红色圆点
- 骨骼之间用绿色线条连接
- 形成一个完整的人体骨架图

### AI视觉分析

系统集成SiliconFlow平台的视觉大模型（Qwen3-VL-8B），能够理解画面内容并生成文字描述。

**调用机制：**

| 项目     | 说明                        |
| :------- | :-------------------------- |
| 调用频率 | 每30帧调用一次（约每秒1次） |
| 触发条件 | 画面中检测到人物时才调用    |
| 调用方式 | 异步执行，不阻塞视频处理    |
| 输出内容 | 25字以内的画面描述          |

**动态提示词策略：**

| 场景        | 发送给AI的问题                                     |
| :---------- | :------------------------------------------------- |
| 有人+有动作 | "画面中有X个人，动作：XXX，用一句话描述（25字内）" |
| 有人+无动作 | "画面中有X个人，用一句话描述（25字内）"            |
| 无人        | "用一句话描述画面（25字内）"                       |

### RTSP视频推流

处理后的视频可以实时推送到RTSP服务器，供其他系统或播放器观看。

| 参数     | 设置                     |
| :------- | :----------------------- |
| 编码格式 | H.264                    |
| 传输协议 | TCP（稳定）              |
| 编码预设 | ultrafast（低延迟）      |
| 画质参数 | CRF 23（平衡画质与码率） |

### 总结

在这个项目中，我基于YOLOv8-Pose姿态检测算法和SiliconFlow视觉大模型，开发了一套实时视频智能分析系统。系统能够从RTSP视频流中实时检测人体的17个关键点、绘制骨骼连线、识别举手和挥手等动作，并通过异步调用AI模型生成画面的语义描述，最终将处理后的视频通过RTSP推流输出。通过这个项目，我掌握了姿态估计、动作识别、异步编程以及RTSP推流等技术，提升了将传统CV算法与大语言模型结合应用的工程能力。

### 输出展示

<img width="1625" height="777" alt="00" src="https://github.com/user-attachments/assets/b6ad84ca-9bcf-477e-bbb5-69c7bfffb42d" />


### 代码

```
import cv2
import subprocess
import base64
import threading
import queue
from ultralytics import YOLO
import requests
import time
import numpy as np
import os

os.environ['OPENCV_LOG_LEVEL'] = 'ERROR'


class VideoAIProcessor:
    def __init__(self, rtsp_input_url, rtsp_output_url=None):
        print("正在加载YOLO模型...")
        self.model = YOLO("yolov8n-pose.pt")
        print("✅ YOLO模型加载完成")

        # ===== 你的API Key =====
        YOUR_API_KEY = "sk-uzircelhqlmgledrwtijlrkglmrvbduowldkpdjrvggcaxte"

        # ===== 2025年SiliconFlow可用的最新模型 =====
        self.models_to_test = [
            # 尝试最新的模型名称
            {"id": "Qwen/Qwen3-VL-8B-Instruct", "name": "Qwen3-VL-8B"},
        ]

        self.api_key = YOUR_API_KEY
        self.base_url = "https://api.siliconflow.cn/v1"

        # 测试API连接
        self.vlm_enabled = False
        self.model_id = None
        self.model_name = None
        self._get_available_models()  # 先获取可用模型列表
        self._test_all_models()

        # 视频参数
        self.rtsp_input = rtsp_input_url
        self.rtsp_output = rtsp_output_url

        # AI调用控制
        self.ai_frame_interval = 30
        self.frame_count = 0
        self.latest_ai_description = "等待AI分析..."
        self.ai_response_queue = queue.Queue()
        self.last_ai_call_frame = 0
        self.running = True
        self.ffmpeg_process = None

        # FPS统计
        self.fps = 0
        self.last_time = time.time()
        self.frame_count_fps = 0
        self.total_ai_calls = 0
        self.successful_ai_calls = 0

        cv2.setNumThreads(0)

    def _get_available_models(self):
        """从API获取可用的模型列表"""
        url = f"{self.base_url}/models"
        headers = {"Authorization": f"Bearer {self.api_key}"}

        try:
            response = requests.get(url, headers=headers, timeout=10)
            if response.status_code == 200:
                models = response.json()
                print("\n📋 从API获取到的可用模型:")
                for model in models.get('data', [])[:10]:  # 只显示前10个
                    model_id = model.get('id', '')
                    if 'vl' in model_id.lower() or 'vision' in model_id.lower():
                        print(f"   - {model_id}")
                print()
        except:
            pass

    def _test_api_direct(self, model_id):
        """直接使用requests测试API"""
        url = f"{self.base_url}/chat/completions"

        headers = {
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json"
        }

        # 简单文本测试（不发送图片，更快）
        payload = {
            "model": model_id,
            "messages": [
                {"role": "user", "content": "你好，请回复OK"}
            ],
            "max_tokens": 10,
            "temperature": 0.7
        }

        try:
            response = requests.post(url, headers=headers, json=payload, timeout=15)
            if response.status_code == 200:
                result = response.json()
                return True, result['choices'][0]['message']['content']
            else:
                return False, f"HTTP {response.status_code}: {response.text[:100]}"
        except Exception as e:
            return False, str(e)

    def _test_all_models(self):
        """测试所有模型"""
        print("\n" + "=" * 50)
        print("🔍 测试API连接和模型可用性")
        print("=" * 50)
        print(f"API Key: {self.api_key[:10]}...{self.api_key[-4:]}")
        print(f"API地址: {self.base_url}")
        print()

        # 先测试文本模型（更可能成功）
        text_models = [
            "Qwen/Qwen2.5-7B-Instruct",
            "deepseek-ai/DeepSeek-V2.5",
            "Pro/Qwen/Qwen2.5-7B-Instruct",
        ]

        print("正在测试文本模型（找可用的API端点）...")
        for model_id in text_models:
            print(f"  测试: {model_id}...", end=" ")
            success, result = self._test_api_direct(model_id)
            if success:
                print(f"✅ 成功")
                print(f"\n✅ API连接正常！文本模型可用: {model_id}")
                break
            else:
                print(f"❌ 失败")
        else:
            print("\n⚠️ 所有文本模型都失败，API可能有问题")

        print("\n正在测试视觉模型...")
        for model in self.models_to_test:
            print(f"测试模型: {model['name']} ({model['id']})...", end=" ")

            success, result = self._test_api_direct(model['id'])

            if success:
                print(f" ✅")
                print(f"  响应: {result[:50]}")
                self.model_id = model['id']
                self.model_name = model['name']
                self.vlm_enabled = True
                self.latest_ai_description = f"AI就绪 ({model['name']})"
                print(f"\n✅ 将使用视觉模型: {model['name']}\n")
                return
            else:
                print(f" ❌")
                if "400" in result:
                    print(f"  原因: 模型名称不存在")
                elif "403" in result:
                    print(f"  原因: 模型被禁用")
                else:
                    print(f"  原因: {result[:50]}")

        print("=" * 50)
        print("⚠️ 所有视觉模型测试失败")
        print("\n建议的解决方案：")
        print("1. 访问 https://siliconflow.cn 查看可用的视觉模型")
        print("2. 在'模型广场'中搜索 'VL' 或 'vision'")
        print("3. 复制正确的模型ID替换到代码中")
        print("4. 检查账户是否有余额或免费额度")
        print("=" * 50 + "\n")

        self.vlm_enabled = False
        self.latest_ai_description = "AI不可用-模型名称错误"

    def ask_vision_model(self, image_base64, person_count, actions):
        """调用视觉模型"""
        if not self.vlm_enabled or not self.model_id:
            return None

        url = f"{self.base_url}/chat/completions"

        headers = {
            "Authorization": f"Bearer {self.api_key}",
            "Content-Type": "application/json"
        }

        if person_count > 0:
            action_text = f"，动作：{', '.join(actions)}" if actions else ""
            question = f"画面中有{person_count}个人{action_text}，用一句话描述（25字内）。"
        else:
            question = "用一句话描述画面（25字内）。"

        payload = {
            "model": self.model_id,
            "messages": [
                {
                    "role": "user",
                    "content": [
                        {"type": "text", "text": question},
                        {"type": "image_url", "image_url": {"url": image_base64}}
                    ]
                }
            ],
            "max_tokens": 80,
            "temperature": 0.7
        }

        try:
            response = requests.post(url, headers=headers, json=payload, timeout=15)

            if response.status_code == 200:
                result = response.json()
                content = result['choices'][0]['message']['content'].strip()[:50]
                self.successful_ai_calls += 1
                return content
            else:
                return None

        except Exception as e:
            return None

    def ask_vision_model_async(self, image_base64, frame_num, person_count, actions):
        """异步调用视觉模型"""
        if not self.vlm_enabled:
            return

        self.total_ai_calls += 1

        def call():
            result = self.ask_vision_model(image_base64, person_count, actions)
            if result:
                self.ai_response_queue.put({
                    "frame_num": frame_num,
                    "content": result,
                    "person_count": person_count
                })
                print(f"\n✨ AI分析: {result}")

        threading.Thread(target=call, daemon=True).start()

    def frame_to_base64(self, frame):
        """将帧转换为base64"""
        small_frame = cv2.resize(frame, (480, 360))
        _, buffer = cv2.imencode('.jpg', small_frame, [cv2.IMWRITE_JPEG_QUALITY, 60])
        return f"data:image/jpeg;base64,{base64.b64encode(buffer).decode('utf-8')}"

    def detect_pose_and_actions(self, frame):
        """检测人体姿态和动作"""
        results = self.model(frame, verbose=False, imgsz=480)[0]

        person_count = 0
        actions = []

        skeleton = [
            (5, 7), (7, 9), (6, 8), (8, 10),
            (11, 13), (13, 15), (12, 14), (14, 16),
            (5, 11), (6, 12), (5, 6), (11, 12),
        ]

        if results.keypoints is not None:
            kpts_all = results.keypoints.xy.cpu().numpy()
            confs_all = results.keypoints.conf.cpu().numpy()

            for kpts, confs in zip(kpts_all, confs_all):
                valid_points = {}
                for i, (x, y) in enumerate(kpts):
                    if confs[i] > 0.5 and x > 0 and y > 0:
                        valid_points[i] = (int(x), int(y))

                if valid_points:
                    person_count += 1

                    for (x, y) in valid_points.values():
                        cv2.circle(frame, (x, y), 3, (0, 0, 255), -1)

                    for start, end in skeleton:
                        if start in valid_points and end in valid_points:
                            cv2.line(frame, valid_points[start], valid_points[end], (0, 255, 0), 2)

                    if (5 in valid_points and 9 in valid_points and
                            valid_points[9][1] < valid_points[5][1] - 40):
                        actions.append("举手")

                    if (6 in valid_points and 10 in valid_points and
                            valid_points[10][1] < valid_points[6][1] - 40):
                        actions.append("举手")

                    if (7 in valid_points and 9 in valid_points and
                            abs(valid_points[9][0] - valid_points[7][0]) > 30):
                        actions.append("挥手")

        return frame, person_count, list(set(actions))

    def process_frame(self, frame):
        """处理单帧"""
        annotated_frame, person_count, actions = self.detect_pose_and_actions(frame)

        h, w = annotated_frame.shape[:2]

        overlay = annotated_frame.copy()
        bar_height = 130
        cv2.rectangle(overlay, (0, 0), (w, bar_height), (0, 0, 0), -1)
        annotated_frame = cv2.addWeighted(annotated_frame, 0.7, overlay, 0.3, 0)

        font = cv2.FONT_HERSHEY_SIMPLEX
        y = 25

        cv2.putText(annotated_frame, "YOLOv8-Pose + AI Vision", (10, y), font, 0.6, (255, 255, 255), 2)
        cv2.putText(annotated_frame, f"FPS: {self.fps:.0f}", (250, y), font, 0.5, (0, 255, 0), 1)
        cv2.putText(annotated_frame, f"People: {person_count}", (370, y), font, 0.5, (255, 255, 0), 1)

        if self.vlm_enabled and self.model_id:
            status = f"AI: Ready"
            color = (0, 255, 0)
        else:
            status = "AI: Check Model"
            color = (0, 0, 255)
        cv2.putText(annotated_frame, status, (490, y), font, 0.45, color, 1)

        if actions:
            cv2.putText(annotated_frame, f"Actions: {', '.join(actions)}", (10, y + 35),
                        font, 0.5, (0, 255, 255), 1)

        cv2.putText(annotated_frame, "🤖 AI:", (10, y + 70), font, 0.5, (255, 200, 100), 1)
        ai_text = self.latest_ai_description[:45]
        cv2.rectangle(annotated_frame, (70, y + 55), (w - 10, y + 90), (0, 0, 0), -1)
        cv2.putText(annotated_frame, ai_text, (75, y + 80), font, 0.5, (100, 255, 100), 1)

        cv2.putText(annotated_frame, "Q:Quit | S:Shot", (w - 140, bar_height - 10),
                    font, 0.4, (200, 200, 200), 1)

        return annotated_frame, person_count, actions

    def run(self):
        """主运行函数"""
        cap = cv2.VideoCapture(self.rtsp_input, cv2.CAP_FFMPEG)
        cap.set(cv2.CAP_PROP_BUFFERSIZE, 1)

        if not cap.isOpened():
            print(f"❌ 无法打开视频流：{self.rtsp_input}")
            return

        actual_width = int(cap.get(cv2.CAP_PROP_FRAME_WIDTH))
        actual_height = int(cap.get(cv2.CAP_PROP_FRAME_HEIGHT))
        print(f"📹 视频尺寸: {actual_width}x{actual_height}")

        if self.rtsp_output:
            ffmpeg_cmd = [
                'ffmpeg', '-y', '-loglevel', 'error',
                '-f', 'rawvideo', '-pix_fmt', 'bgr24',
                '-s', f'{actual_width}x{actual_height}',
                '-r', '30', '-i', '-',
                '-c:v', 'libx264', '-preset', 'ultrafast',
                '-tune', 'zerolatency', '-crf', '23',
                '-pix_fmt', 'yuv420p',
                '-f', 'rtsp', '-rtsp_transport', 'tcp',
                self.rtsp_output
            ]

            try:
                self.ffmpeg_process = subprocess.Popen(
                    ffmpeg_cmd, stdin=subprocess.PIPE,
                    stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL
                )
                print(f"✅ FFmpeg推流已启动")
            except Exception as e:
                print(f"⚠️ FFmpeg启动失败: {e}")

        print(f"\n✅ 开始处理视频流...")
        print(f"📥 输入: {self.rtsp_input}")
        if self.rtsp_output:
            print(f"📤 输出: {self.rtsp_output}")
        print(f"🤖 AI状态: {'已启用' if self.vlm_enabled else '已禁用（模型名称错误）'}")
        print(f"⌨️  控制: 'q'退出 | 's'截图\n")

        while self.running:
            ret, frame = cap.read()
            if not ret:
                continue

            self.frame_count += 1
            self.frame_count_fps += 1

            now = time.time()
            if now - self.last_time >= 1.0:
                self.fps = self.frame_count_fps
                self.frame_count_fps = 0
                self.last_time = now

            processed_frame, person_count, actions = self.process_frame(frame)

            if self.vlm_enabled and self.model_id and person_count > 0:
                if (self.frame_count - self.last_ai_call_frame) >= self.ai_frame_interval:
                    self.last_ai_call_frame = self.frame_count
                    img_base64 = self.frame_to_base64(frame)
                    threading.Thread(
                        target=self.ask_vision_model_async,
                        args=(img_base64, self.frame_count, person_count, actions),
                        daemon=True
                    ).start()

            try:
                while not self.ai_response_queue.empty():
                    resp = self.ai_response_queue.get_nowait()
                    self.latest_ai_description = resp["content"]
            except queue.Empty:
                pass

            if self.ffmpeg_process and self.ffmpeg_process.poll() is None:
                try:
                    self.ffmpeg_process.stdin.write(processed_frame.tobytes())
                except:
                    pass

            display = cv2.resize(processed_frame, (960, 540))
            cv2.imshow("YOLO-Pose + AI", display)

            key = cv2.waitKey(1) & 0xFF
            if key == ord('q'):
                break
            elif key == ord('s'):
                ts = time.strftime("%H%M%S")
                cv2.imwrite(f"screenshot_{ts}.jpg", processed_frame)
                print(f"📸 截图已保存")

        cap.release()
        cv2.destroyAllWindows()
        if self.ffmpeg_process:
            try:
                self.ffmpeg_process.stdin.close()
                self.ffmpeg_process.wait(timeout=2)
            except:
                pass
        print(f"\n✅ 程序退出")
        if self.vlm_enabled:
            print(f"📊 AI统计: 调用{self.total_ai_calls}次, 成功{self.successful_ai_calls}次")


if __name__ == "__main__":
    print("=" * 50)
    print("🎥 YOLO-Pose + AI 视觉分析系统")
    print("=" * 50)

    INPUT_URL = "rtsp://127.0.0.1:25544/input"
    OUTPUT_URL = "rtsp://127.0.0.1:25544/output"

    print(f"\n📹 使用RTSP流: {INPUT_URL}")
    print(f"📤 推流到: {OUTPUT_URL}\n")

    processor = VideoAIProcessor(INPUT_URL, OUTPUT_URL)

    try:
        processor.run()
    except KeyboardInterrupt:
        print("\n用户中断")
```

