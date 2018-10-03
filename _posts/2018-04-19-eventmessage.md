---
layout:     post
title:      "替代SendMessage反射机制的方案"
subtitle:   "事件驱动程序设计"
date:       2018-10-03 12:00:00
author:     "LYQ"
header-img: "img/in-post/default-bg.jpg"
tags:
    - 开发日志
    - 独立游戏开发
    - Unity
---

## 写在前面

SendMessage虽然简单易用且可读性高，但其过度依赖反射机制，对性能的影响很大。利用事件驱动程序设计，可以实现与其一样的效果，且执行所需时间更少。

## 效率的比较

![微信截图_20181003105026.png](https://upload-images.jianshu.io/upload_images/11723713-62289e558d6e0b87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## EventManage(single design pattern)

````
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.AI;

public class EventManager : MonoBehaviour {

	#region enum
	//-------------------------------------------
	//enum defining all possible game events
    public enum EVENT_TYPE
	{
		FSM_ENTER,
		FSM_EXIT,
		FSM_UPDATE
	}
	
	#endregion
	
 

	#region variables
    //-------------------------------------------
	//singleton design pattern
	private static EventManager instance = null;
	
	//Array of listener object
	public Dictionary<EVENT_TYPE,List<OnEvent>> Listeners = new Dictionary<EVENT_TYPE, List<OnEvent>>();

	//Defining a delegate type for events
	public delegate void OnEvent(EVENT_TYPE Event_Type, Component Sender = null, object Param = null);
	
    #endregion
	
	
		
    #region Properties Setting
	//-------------------------------------------
    //public access to instace
    public static EventManager Instance
	{
		get { return instance; }
	}

	#endregion
	
	

	private void Awake()
	{
		//If no instance exists,then assign this instance
		if (instance == null)
		{
			instance = this;
			DontDestroyOnLoad(this);
		}
		else
		{
			DestroyImmediate(this);
		}
	}
	//-------------------------------------------

	
	
	/// <summary>
	/// Add listener_object to array of listeners
	/// </summary>
	/// <param name="Event_Type">Event to Listen</param>
	/// <param name="Listener">Object to Listen for event</param>
	public void AddListener(EVENT_TYPE Event_Type, OnEvent Listener)
	{
		List<OnEvent> ListenerList = null;
		if (Listeners.TryGetValue(Event_Type, out ListenerList))
		{
			ListenerList.Add(Listener);
			return;
		}
		
		ListenerList = new List<OnEvent>();
		ListenerList.Add(Listener);
		Listeners.Add(Event_Type, ListenerList);
	}
	//-------------------------------------------

	
	
	/// <summary>
	///Funiction to post event to listeners 
	/// </summary>
	/// <param name="Event_Type">event to invoke</param>
	/// <param name="Sender">Object of invokeing event</param>
	/// <param name="Param">Optional argument</param>
	public void PostNotification(EVENT_TYPE Event_Type, Component Sender = null, object Param = null)
	{
		List<OnEvent> ListenerList = null;
		
		//If no entry exists, then exit
		if (!Listeners.TryGetValue(Event_Type, out ListenerList))
		{
			return;
		}

		for (int i = 0; i < ListenerList.Count; i++)
		{
			if (!ListenerList[i].Equals(null))
				ListenerList[i](Event_Type, Sender, Param);
		}
	}

	 



}

````

定义EventMessage,将其作为一个持久性的单例对象。该类绑定在场景的空对象上。                    

## 注册与调用

* 注册示例代码
````
 EventManager.Instance.AddListener(EventManager.EVENT_TYPE.FSM_ENTER, EnterEvent); //start()
//=====================================================================================
public void EnterEvent(EventManager.EVENT_TYPE Event_Type, Component Sender, object Param = null)
    {
        if(Sender.name.Equals("Player"))
            switch ((string) Param)
            {
                case "Jump":
                    OnjumpEnter();
                    break;
                case "Ground":
                    OnGroundEnter();
                    break;
                case "Jab":
                    OnJabEnter();
                    break;
                case "General":
                    OnGeneralEnter();
                    break;
                default:
                    Debug.Log("EnterEvent Error");
                    break;
            }
    }
````

* 调用示例代码

````
public string[] OnEnterMessage; 

override public void OnStateEnter(Animator animator, AnimatorStateInfo stateInfo, int layerIndex)
	{
		foreach (string signal in OnEnterMessage)
		{
			EventManager.Instance.PostNotification(EventManager.EVENT_TYPE.FSM_ENTER, animator.transform.parent, signal);
		}

		foreach (string signal in EnterClearSignal)
		{
			animator.ResetTrigger(signal);
		}

	}
````