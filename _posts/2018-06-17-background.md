---
layout:     post
title:      "无限滚动背景图的两种方法"
subtitle:   "循环背景的实现（unity）"
date:       2018-06-17 12:00:00
author:     "LYQ"
header-img: "img/in-post/default-bg.jpg"
tags:
    - Unity
    - 2D游戏
    - 项目笔记
---

## 写在前面

一个游戏，如果将所需要的场景背景都做到一个模型中，会导致数据量非常大，而且还得在游戏开始的时候就生成这些背景，非常麻烦。背景仅仅用于显示，与游戏内容没有多大的关系，背景的显示仅局限与角色周围的一小部分，所以循环使用背景资源非常重要。

 
## 实现代码

* 第一种方法

````
    public const float WIDTH = 10f;

    public const int MODEL_NUM = 3;

    [SerializeField] private GameObject main_camera;

    private Vector3 initPos;

    void Start()
    {
        initPos = this.transform.position;
    }

        float total_width = WIDTH * MODEL_NUM;

        Vector3 floor_Pos = this.transform.position;

        Vector3 camera_Pos = this.main_camera.transform.position;

        if (floor_Pos.x + total_width / 2 < camera_Pos.x)
        {
            floor_Pos.x += total_width;

            this.transform.position = floor_Pos;
        }

        if (camera_Pos.x < floor_Pos.x - total_width / 2)
        {
            floor_Pos.x -= total_width;
            this.transform.position = floor_Pos;
        }
````
当摄像机超过背景图位置+总长度的中间点时，将位置更换。

* 第二种方法

````
float total_width = WIDTH * MODEL_NUM;

        Vector3 camera_position = this.main_camera.transform.position;


        float dist = camera_position.x - this.initPos.x;


        // 模型出现在total_width 的整数倍位置
        // 用初始位置的距离除以整体背景的宽度，再四舍五入

        int n = Mathf.RoundToInt(dist / total_width);

        Vector3 position = this.initPos;

		position.x += n*total_width;

		this.transform.position = position;
````
计算相机的位置与初始位置的差值，将其与total_width相除得到n值，根据n值来滚动背景图，此方法不会因为角色移速导致引起背景消失。

---

## 关于数学类的RoundToInt以及FloorToInt

* Mathf.FloorToInt(向下取整)

````
// Prints 10
Debug.Log(Mathf.FloorToInt(10.0));
// Prints 10
Debug.Log(Mathf.FloorToInt(10.2));
// Prints 10
Debug.Log(Mathf.FloorToInt(10.7));
// Prints -10
Debug.Log(Mathf.FloorToInt(-10.0));
// Prints -11
Debug.Log(Mathf.FloorToInt(-10.2));
// Prints -11
Debug.Log(Mathf.FloorToInt(-10.7));
````

* Mathf.RoundToInt(四舍五入)

````
 
// Prints 10
Debug.Log(Mathf.RoundToInt(10.0));
// Prints 10
Debug.Log(Mathf.RoundToInt(10.2));
// Prints 11
Debug.Log(Mathf.RoundToInt(10.7));
// Prints 10
Debug.Log(Mathf.RoundToInt(10.5));
// Prints 12
Debug.Log(Mathf.RoundToInt(11.5));
// Prints -10
Debug.Log(Mathf.RoundToInt(-10.0));
// Prints -10
Debug.Log(Mathf.RoundToInt(-10.2));
// Prints -11
Debug.Log(Mathf.RoundToInt(-10.7));
// Prints -10
Debug.Log(Mathf.RoundToInt(-10.5));
// Prints -12
Debug.Log(Mathf.RoundToInt(-11.5));
 

````
如果数字末尾是.5，因此它是在两个整数中间，不管是偶数或是奇数，将返回偶数。