---
layout:     post
title:      "Unity播放视频以及跳过视频"
subtitle:   "NGUI"
date:       2018-05-13 12:00:00
author:     "LYQ"
header-img: "img/in-post/default-bg.jpg"
tags:
    - Unity
    - 读书笔记
    - NGUI
---

本文记录了unity实现播放视频以及跳过视频的代码，以便日后查阅。

## 源码

 ````
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
 

public class MovieControl : MonoBehaviour
{ 
    //是否跳过动画的提示信息
    public UILabel TipManager;

    //跳过动画所需时间的进度条
    public UISlider SkipSlider;

	MovieTexture VideoSource;

    //是否展示提示信息
	bool IsShowTipManager;

    //提示信息存在时间
    float TipShowTime;

    //按住跳过所需时间
    float HoldTime;
	 
	void Start () {

        TipManager.text = "Press on 'ESC' skip";
        TipShowTime = 0.0f;
        HoldTime = 0.0f;
        SkipSlider.value = 0;
		IsShowTipManager = false;
	    VideoSource = (MovieTexture)this.GetComponent<UITexture> ().mainTexture;
		VideoSource.loop = false;
		VideoSource.Play ();	 
	}
	
	 
	void Update () {

        //按下回车或ESC或空格显示跳过画面的提示信息
        if (Input.GetKeyDown(KeyCode.Escape) || Input.GetKeyDown(KeyCode.Space) || Input.GetKeyDown(KeyCode.Return))
        {
            IsShowTipManager = true;
        }

        
        if (IsShowTipManager && TipShowTime < 6f)
        {
            TipShowTime += Time.deltaTime;
            ActiveTipManager(true);  

            //在提示UI出现的情况下按住ESC键3秒可以停止动画
            if (Input.GetKey(KeyCode.Escape)) 
            {
                //按下ESC键则重置提示UI的存在时间
                TipShowTime = 0.0f;

                HoldTime += Time.deltaTime;
                
                //将按住时间绑定进度条
                SkipSlider.value = (HoldTime / 3);

                //按住超过3秒
                if (HoldTime >= 3f)
                {
                    VideoSource.Stop();
                    IsShowTipManager = false;
                    ActiveTipManager(false);
                }
            }
            else
            {
                //松开ESC键重置按住时间
                HoldTime = 0.0f;

                //松开ESC键回滚进度条
                SkipSlider.value = 0;
                
            }
        }

        else
        {
            ActiveTipManager(false);
            IsShowTipManager = false;
            TipShowTime = 0.0f;
        }
	}

    /// <summary>
    /// 是否激活提示UI元素
    /// </summary>
    /// <param name="BoolStr"></param>
    void ActiveTipManager(bool BoolStr)
    {
         
        if (BoolStr)
        {
            TipManager.gameObject.SetActive(true);
            SkipSlider.gameObject.SetActive(true);
        }

        else if(BoolStr == false)
        {
            TipManager.gameObject.SetActive(false);
            SkipSlider.gameObject.SetActive(false);
        }

         

    }

   



}

````

## 实际效果
![Video.gif](https://upload-images.jianshu.io/upload_images/11723713-7db3f6845c6d0559.gif?imageMogr2/auto-orient/strip)

























