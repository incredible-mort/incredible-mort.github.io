---
title: Get Start With My Website
date: 2023-08-22 17:30:00 +/-TTTT
author: <author_id>
categories: [Article, Entertainment]
tags: [play]     # TAG names should always be lowercase
---
## 摘要

OAK-D PRO W 的相机录制与标定工作

OAK-D PRO W 在PC与RK3588端的算法运行情况

RealSense T265 的路径可靠性功能调查

## RealSense T265

> [Intel RealSense Documentation Code-overview](https://dev.intelrealsense.com/docs/rs-pose-predict#code-overview)

​		这部分工作主要围绕着机器人部门在智能喷枪规划中，对于T265设备的标定与算法测试之外，T265在自身所在的SDK包中，具备标出VIO路径为绿色、黄色、红色的功能，并且在设备运行过程中，偶然会出现设备端报错Slam error speed，对T265设备的自身使用有无注意事项进行调研。

​		在realsense2包中创建了一个结构体叫做 tracked_point，而这个结构体中的属性confidence在后续在轨迹绘制的方法中，被作为判断轨迹绘制颜色的依据，以此为起点进行查找

```c++
struct tracked_point
{
    rs2_vector point;
    unsigned int confidence;
};
```

```c++
void draw_trajectory()
{
    float3 colors[]{
        { 0.7f, 0.7f, 0.7f },
        { 1.0f, 0.0f, 0.0f },
        { 1.0f, 1.0f, 0.0f },
        { 0.0f, 1.0f, 0.0f },
    };

    glLineWidth(2.0f);
    glBegin(GL_LINE_STRIP);
    for (auto&& v : trajectory)
    {
        auto c = colors[v.confidence];
        glColor3f(c.x, c.y, c.z);
        glVertex3f(v.point.x, v.point.y, v.point.z);
    }
    glEnd();
    glLineWidth(0.5f);
}
```

​		以此为起点需要知道，库中的代码在什么时候获取到了这个结构体名为追踪点的信息，并对其中的confidence属性进行了赋值，找到了他的上一级计算节点为

```c++
// From the matrix we found, get the new location point
rs2_vector tr{ r[12], r[13], r[14] };
// Create a new point to be added to the trajectory
tracked_point p{ tr , pose_data.tracker_confidence };
// Register the new point
tracker.add_to_trajectory(p);
```

​		所以只要知道pose_data的数据来源，就能够找到tracked_point的confidence数据来源

```c++
	// Wait for the next set of frames from the camera
    auto frames = pipe.wait_for_frames();
    // Get a frame from the pose stream
    auto f = frames.first_or_default(RS2_STREAM_POSE);
    // Cast the frame to pose_frame and get its data
    auto pose_data = f.as<rs2::pose_frame>().get_pose_data();
```

​		而==pose_data直接来自于相机中获取的帧数据==中，并不是在后续的算法中进行评估，而是设备在录制的过程中通过硬件状态评估了数据可信度
