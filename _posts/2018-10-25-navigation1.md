---
layout:     post
title:      "Navigation自动寻路技术"
subtitle:   "第二篇：代理寻路测试"
date:       2018-10-25 12:00:00
author:     "Lyq"
header-img: "img/in-post/default-bg.jpg"
tags:
    - Unity
    - Editor
    - Navigation
---

## 写在前面

>**在场景中创建一个圆柱体，将碰撞体删除，添加NavMeshAgent组件**

## 正文

在圆柱体上挂上以下代码

````
using System.Collections;
using System.Collections.Generic;
using System.Runtime.InteropServices;
using UnityEngine;
using UnityEngine.AI;

/// <summary>
/// 代理寻路案例
/// </summary>
[RequireComponent(typeof(NavMeshAgent))]
public class NavMeshAgentExample : MonoBehaviour
{
	//代理组件
	private NavMeshAgent _agent;
	public AIWallpointNetWork WallpointNetWork;
	//当前初始路径点
	public int CurrentIndex;
	
	//当前路径是否已经过时
	[SerializeField]private bool _isPathStale;
	//是否存在路径
	[SerializeField]private bool _hasPath;
	//是否在计算路径，该路径未完成
	[SerializeField]private bool _pathPending;
	//当前路径状态
	public NavMeshPathStatus PathStatus;
	
	private void Awake()
	{
		_agent = GetComponent<NavMeshAgent>();
	}

	private void Start()
	{
		if (!WallpointNetWork){
			Debug.Log("无法获取到路径列表");
			return;
		}
		SetNextTarget(false);
	}

	/// <summary>
	/// 设置下一个路径点
	/// </summary>
	/// <param name="isIncrease">是否切换到下个路径点</param>
	public void SetNextTarget(bool isIncrease)
	{
		if (!WallpointNetWork){
			Debug.Log("无法获取到路径列表");
			return;
		}
		//增长步数
		int incStep = isIncrease ? 1 : 0;
		//下一个路径点
		Transform nextTargeTransform = null;

		while (nextTargeTransform == null)
		{
			//解决第一个路径点为空时会出现死循环的Bug
			if (WallpointNetWork.WallPoints[0] == null && CurrentIndex == WallpointNetWork.WallPoints.Count - 1)
			{
				CurrentIndex = 0;
			}
			//下一个路径点下标值得计算
			int nextIndex = (CurrentIndex + incStep >= WallpointNetWork.WallPoints.Count) ? 0 : CurrentIndex + incStep;
			nextTargeTransform = WallpointNetWork.WallPoints[nextIndex];

			//不为空的话作为导航点，空的话遍历下个元素
			if (nextTargeTransform != null)
			{
				_agent.destination = nextTargeTransform.position;
				CurrentIndex = nextIndex;
				return;
			}

			CurrentIndex++;
			
		}
		
	}

	private void Update()
	{
		_hasPath = _agent.hasPath;
		_pathPending = _agent.pathPending;
		_isPathStale = _agent.isPathStale;
		PathStatus = _agent.pathStatus;
		
		
		//在没有路径以及路径计算完成的情况下 || 无效路径 || 无法到达的路径
		if ((!_hasPath && !_pathPending) || PathStatus == NavMeshPathStatus.PathInvalid || PathStatus == NavMeshPathStatus.PathPartial)
		{
			SetNextTarget(true);
		}

		#region 测试用代码，可删除
		Debug.Log(PathStatus);
		if (!_hasPath){
			//Debug.Log("hasPath");
			}
		if (_pathPending){
			//Debug.Log("pathPending");
			}
		#endregion
	}
}

````



 