---
layout:     post
title:      "Navigation自动寻路技术"
subtitle:   "第一篇：测试工具的定制"
date:       2018-10-24 12:20:00
author:     "LYQ"
header-img: "img/in-post/default-bg.jpg"
tags:
    - 游戏AI
    - Unity
    - Editor
---

## 写在前面

Navigation是Unity内置的寻路解决方案

## 场景图

![1](https://upload-images.jianshu.io/upload_images/11723713-20d97a011703cdc7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>**注：蓝色的小旗是路径点。场景已经进行过烘焙，只有烘焙过的场景才能使用Navigation。**

## 正文

 * **将路径点组成一个网络进行管理**

````
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

/// <summary>
/// 将路径点组成一个网络进行管理
/// </summary>
[ExecuteInEditMode]
public class AIWallpointNetWork : MonoBehaviour
{

	//路径的显示类型
	public enum NetWorkDisPlayMode
	{
		//不显示
		NONE,
		//所有路径点连接在一起
		CONNECT,
		//两个路径点组成一条路径
		PATH 
	}

	//默认状态
	[HideInInspector]
	public NetWorkDisPlayMode DisPlayMode = NetWorkDisPlayMode.CONNECT;
	
	//存储路径点的列表
	public List<Transform> WallPoints = new List<Transform>();
	
	//PATH模式下的起始路径点
	[HideInInspector]
	public int from = 0;
	
	//PATH模式下的目标路径点
	[HideInInspector]
	public int to = 1;
}

````

 * **声明编辑器类来定制自己的寻路测试工具**

>**注：请将该编辑器类放在Editor目录下**

````
using System.Collections;
using System.Collections.Generic;
using NUnit.Framework.Constraints;
using UnityEngine;
using UnityEditor;
using UnityEngine.AI;
using UnityEngine.Experimental.PlayerLoop;

/// <summary>
/// 该编辑器类定制自己的寻路测试工具
/// </summary>
[CustomEditor(typeof(AIWallpointNetWork))]
public class AIWallpointNetWorkEditor : Editor
{
	//自定义属性面板
	public override void OnInspectorGUI()
	{
		//所选目标
		AIWallpointNetWork network = target as AIWallpointNetWork;

		//定制枚举变量编辑器样式
		network.DisPlayMode =
			(AIWallpointNetWork.NetWorkDisPlayMode) EditorGUILayout.EnumPopup("DisPlayMode", network.DisPlayMode);
		
		//只有在PATH模式下才能操作from,to
		if (network.DisPlayMode == AIWallpointNetWork.NetWorkDisPlayMode.PATH)
		{
			//定制滑动条
			network.from = EditorGUILayout.IntSlider("Start", network.from, 0, network.WallPoints.Count - 1);
			network.to = EditorGUILayout.IntSlider("End", network.to, 0, network.WallPoints.Count - 1);
		}

		//绘制出原有默认的属性面板，当然已经声明了[HideInInspector]的字段不会被绘制，而是进行自定义绘制
		DrawDefaultInspector();
	}

	//自定义场景显示
	private void OnSceneGUI()
	{
		AIWallpointNetWork network = target as AIWallpointNetWork;

		//给所有的路径点在场景中显示名称
		for (int i = 0; i < network.WallPoints.Count; i++)
		{
			if (network.WallPoints[i] != null)
				Handles.Label(network.WallPoints[i].transform.position, "WallPoint" + i.ToString());
		}

		//该数组存储所有路径点的位置信息，由于要实现环形连接，所以要加多一个长度
		Vector3[] LineWallPoints = new Vector3[network.WallPoints.Count + 1];

		//环形连接模式
		if (network.DisPlayMode == AIWallpointNetWork.NetWorkDisPlayMode.CONNECT)
		{
			for (int i = 0; i <= network.WallPoints.Count; i++)
			{
				//最后一个值归零，也就是连接回原点
				int index = i != network.WallPoints.Count ? i : 0;
				if (network.WallPoints[index] != null)
				{
					LineWallPoints[i] = network.WallPoints[index].position;
				}
				else
				{
					LineWallPoints[i] = new Vector3(Mathf.Infinity, Mathf.Infinity, Mathf.Infinity);
				}
			}
			//颜色定制
			Handles.color = Color.yellow;
			//开始绘制线条
			Handles.DrawPolyLine(LineWallPoints);
		}

		//PATH模式下，所选的两个路经点会自动绘制出最佳路径，而不是单纯的直线连接
		else
		if(network.DisPlayMode == AIWallpointNetWork.NetWorkDisPlayMode.PATH)
		{
			//导航系统计算的路径
			NavMeshPath Path = new NavMeshPath();

			if (network.WallPoints[network.from] != null && network.WallPoints[network.to] != null)
			{
				//起始点的位置
				Vector3 from = network.WallPoints[network.from].position;
				//目标点的位置
				Vector3 to = network.WallPoints[network.to].position;
				//开始路径计算，路径点存储在corners里
				NavMesh.CalculatePath(from, to, NavMesh.AllAreas, Path);
			}

			//颜色定制，不同模式颜色最好不一样，便于识别
			Handles.color = Color.cyan;
			//开始绘制路线
			Handles.DrawPolyLine(Path.corners);
		}

		
	}
}

````

## 效果图

![微信截图_20181024105328.png](https://upload-images.jianshu.io/upload_images/11723713-ca6247d0d8fa4481.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

![微信截图_20181024105414.png](https://upload-images.jianshu.io/upload_images/11723713-7365ec1366a595ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)




