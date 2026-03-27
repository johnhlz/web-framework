# Sensor Platform (FastAPI + SQLite + Vue 3 + Vite)

工业检测数据接收与可视化监控项目。

## 功能概览
- 后端 API（FastAPI + SQLite + SQLAlchemy）
  - `POST /api/sensor/upload`
  - `GET /api/sensor/latest`
  - `GET /api/sensor/history?limit=100`
  - 自动保留最近 100 条数据
- 前端大屏（Vue 3 + Vite + ECharts）
  - 最新数据卡片
  - 空气质量展示
  - 振动波形图
  - 历史趋势图
  - 每 2 秒自动刷新
- C 模拟上传程序（libcurl）
  - 封装函数供外部主函数调用
  - 自动生成 ISO 8601 时间戳
  - POST 上传到后端

## 项目结构
```text
sensor/
  app/
    api/
      routes_sensor.py
    core/
      config.py
    db/
      base.py
      session.py
    models/
      sensor_data.py
    schemas/
      sensor.py
    main.py
  frontend/
    index.html
    package.json
    vite.config.js
    src/
      main.js
      App.vue
      api.js
      components/
        LayoutHeader.vue
        LatestDataCard.vue
        AirQualityCard.vue
        VibrationChart.vue
        TrendChart.vue
      styles/
        theme.css
  simulator/
    sensor_uploader.h
    sensor_uploader.c
    test_main.c
  requirements.txt
  README.md
```

## 环境准备（Conda）
```powershell
conda activate sensor-backend
```

如未创建环境：
```powershell
conda create -n sensor-backend python=3.12 -y
conda activate sensor-backend
conda install nodejs -y
```

## 启动后端
```powershell
cd /d e:\code\sensor
pip install -r requirements.txt
uvicorn app.main:app --reload
```

后端文档：`http://127.0.0.1:8000/docs`

## 启动前端
```powershell
conda activate sensor-backend
cd /d e:\code\sensor\frontend
npm run dev
```

前端地址：`http://127.0.0.1:5173`

## C 函数用法（主函数中调用）
头文件中已暴露两个接口：
- `sensor_upload_post(...)`：按结构体参数直接上传
- `sensor_upload_interactive(...)`：交互输入并上传

示例（推荐 `sensor_upload_post`）：
```c
#include <stdio.h>
#include "sensor_uploader.h"

int main(void) {
    SensorUploadInput input = {
        .sensor_id = "SENSOR-MAIN-01",
        .imu_x = 0.12,
        .imu_y = -0.23,
        .imu_z = 0.98,
        .air_pm25 = 18.6,
        .air_pm10 = 31.2,
        .air_co2 = 512.4,
        .air_voc = 0.42,
        .temperature = 26.3,
        .humidity = 47.8,
        .vibration_waveform_csv = "0.1,0.2,0.15,-0.05",
        .device_status = "online",
        .quality_result = "normal",
        .remark = "from host main",
        .url = NULL
    };

    long http_status = 0;
    char response[2048];
    int rc = sensor_upload_post(&input, &http_status, response, sizeof(response));

    printf("rc=%d, http=%ld\n", rc, http_status);
    if (response[0] != '\0') {
        printf("resp=%s\n", response);
    }

    return rc;
}
```

编译（Linux）：
```bash
gcc -O2 -Wall -Wextra -std=c11 your_main.c sensor_uploader.c -o app -lcurl
```

使用现成测试主函数（20 组数据）：
```bash
gcc -O2 -Wall -Wextra -std=c11 test_main.c sensor_uploader.c -o test_uploader -lcurl
./test_uploader
```

## 说明
- 后端 Python 依赖维护在 `requirements.txt`。
- 前端依赖维护在 `frontend/package.json`。
- C 程序依赖系统 `libcurl` 开发库，不在 Python requirements 中管理。
