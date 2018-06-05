---
layout:     post
title:      "Unity动画曲线在代码中的应用"
subtitle:   "NGUI"
date:       2018-06-05 12:00:00
author:     "LYQ"
header-img: "img/in-post/default-bg.jpg"
tags:
    - Unity
    - 读书笔记
    - 动画系统
---

本文记录了动态创建动画曲线的实例代码，以便日后查阅。

 

##写在前面
AnimationCurve是Unity3D里一个非常实用的功能。作用是编辑一条任意变化的曲线用在任何你想用在的地方。 如曲线地形，曲线轨迹等。也被用在了模型动画播放时的碰撞盒缩放及重力调节。AnimationCurve 曲线的绘制方法和Ragespline中的物体轮廓勾勒的方法类似。


 
----
##主要代码
````
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.EventSystems;
public class Test01 : MonoBehaviour {
    private AnimationCurve animationCurve;
    private Vector2 initpos;
	private Vector2 destpos;

	bool isAnimation = false;
	void Start () {
		 initpos = this.transform.position;
         destpos = initpos+new Vector2(2,2);
		
	}

   private void Update() {
	    if(Input.GetMouseButton(0))
		{
            Keyframe k1 = new Keyframe(Time.time,0,0,0);
			Keyframe k2 = new Keyframe(Time.time+1f,1,0,0);
			animationCurve = new AnimationCurve(k1,k2);
			isAnimation = true;
		}
   }

	private void LateUpdate() {
       

		if(isAnimation)
		{
        if(Time.time>=animationCurve.keys[animationCurve.length-1].time)
		{
			Debug.Log(Time.time);
			this.transform.position = destpos;
			isAnimation = false;
			return;
		}	
		Vector2 newpos = initpos + (destpos-initpos)*animationCurve.Evaluate(Time.time);
		this.transform.position = newpos;
		}
		
	}


	
	  
}

````
 
















