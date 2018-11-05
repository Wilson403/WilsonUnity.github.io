---
layout:     post
title:      "手柄与键盘之间的动态切换"
subtitle:   "利用状态模式来实现"
date:       2018-11-05 12:00:00
author:     "LYQ"
header-img: "img/in-post/InputState.jpg"
tags:
    - Unity
    - C#
    - 设计模式
---

## 写在前面

简要说明下需求，某游戏同时支持2种控制器，即键鼠以及手柄。游戏默认为手柄操作，当按下键盘或者鼠标的任意一个按键时，会立即切换到键鼠操作。在键鼠模式下，如果按下手柄任意一个按键或者推动手柄操作杆，就又会回到手柄操作。这些功能都是自动完成的，几乎一秒完成，无需玩家自己去手动设置切换。

利用状态模式可以实现这个功能，使用该模式也方便扩展功能，比如说，切换到手柄操作时，把所有操作提示图标都换成手柄键位图标。反之亦然。

本篇博客参考的资料：
——《设计模式与游戏完美开发》 蔡升达

---

## 题外话
最近玩了下《古墓丽影：暗影》，使用手柄在PC上游玩，因为手柄能带来真实的震动反馈，游戏沉浸感更强。但是战斗系统是第三人称射击，用手柄玩射击游戏真的是蛋疼。所以每到枪战是我就用键鼠操作，碰一下鼠标就立即切换，游戏的操作提示图标此时立即替换为键鼠的提示图标。

---

## 什么是状态模式（State）
Gof的解释是：让一个对象随着内部状态的变化而发生变化，就像是换了一个类一样。也就是说，这个对象与外界的对应方式不会发生任何变化。但是内部的‘状态更换’会使对象更换到‘另外一个类’，表现出它在这个状态下所应有的行为。

---

## 状态模式(State)的简单实现
>首先定义一个状态基类，定义一些方法供子类使用，重写。

````
public abstract class State
	{
		protected Context m_Context = null;

		public State(Context theContext)
		{
			m_Context = theContext;
		}			
		public abstract void Handle(int Value);
	}
````

>定义一个控制中心，持有当前状态，能够提供给状态一些信息，协助状态的更换。

````
public class Context
	{
		State	m_State = null;

		public void Request(int Value)
		{
			m_State.Handle(Value);
		}

		public void SetState(State theState )
		{
			Debug.Log ("Context.SetState:" + theState);
			m_State = theState;
		}
	}
````
>状态子类的定义

````
public class ConcreteStateA : State
	{
		public ConcreteStateA( Context theContext):base(theContext)
		{}

		public override void Handle (int Value)
		{
			Debug.Log ("ConcreteStateA.Handle");
			if( Value > 10)
				m_Context.SetState( new ConcreteStateB(m_Context));
		}

	}

````

---

## 手柄键鼠切换实现案例

>由于篇幅有限，暂不介绍随着操作模式的替换同时替换游戏资源（提示图标）

* **首要的问题是，如何确定你按下的按钮是键盘的按钮还是手柄的。我们可以运用Input.anykey来检测所有按钮，然后再利用键值来区分。手柄有部分是轴感应，也要有不同的检测方法**

````
using System;
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class CheckController{

	private static string str = null;
	private static float num = 0;
	
	public static int GetCurController()
	{
		if (Input.anyKeyDown)
		{
			foreach (KeyCode keycode in Enum.GetValues(typeof(KeyCode)))
			{
				if (Input.GetKeyDown(keycode))
				{
					str = keycode.ToString();
					if (str.Length >= 3 && str.Substring(0, 3).Equals("Joy"))
					{
						return -1;
					}
					else
					{
						return 1;
					}
				}
			}
		}

		else
		{
			num = Input.GetAxis("axisX") + Input.GetAxis("axisY") + Input.GetAxis("axis4") + Input.GetAxis("axis5") +
			      Input.GetAxis("LT") + Input.GetAxis("RT") + Input.GetAxis("axis6") + Input.GetAxis("axis7");
			if (num != 0)
			{
				return -1;
			}
		}

		return 0;
	}
}

````

* **代码说明：-1表示手柄模式，1表示键鼠模式。0表示当前模式**

>接下来就可以写实现了，根据上面的一个简单案例，来套入代码

* State基类

````
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class IInputState
{
    #region variable
    protected float Target_Vertical;
    protected float Target_Horizontal;
    protected float Current_Vertical;
    protected float Current_Horizontal;
    protected float Velocity_Vertical;
    protected float Velocity_Horizontal;
    //视角旋转量Y
    public float Jup;
    //视角旋转量Y
    public float Jright;
    //位移量(x,y开方后的值)
    public float DMag;
    //角色控制柄的旋转量
    public Vector3 Dvec;
	//是否奔跑
    public bool isrun;
    //是否防御
    public bool isdefense; 
    //是否锁定敌人
    public bool islock; 
    //是否跳跃
    public bool isjump; 
    public  bool isjumplast;
	//是否攻击
    public  bool isattacklast;
	//是否能够移动
    public bool IsInputEnable = true; 
    //是否前刺
    public bool isFontStab = false;
    //手柄LB键
    public bool lb;
    //手柄LT键
    public bool lt;
    //手柄RB键
    public bool rb;
    //手柄RT键
    public bool rt;
    #endregion
    
//-----------------------------------------------------------
    
    private string m_StateName = "m_StateName";
    public string MStateName
    {
        get { return m_StateName; }
        set { m_StateName = value; }
    }

    protected InputStateController m_Controller = null;
    protected Transform m_transform = null;

    public IInputState(InputStateController mController,Transform trans)
    {
        m_Controller = mController;
        m_transform = trans;
    }

    public virtual void StateBegin()
    {
        
    }

    public virtual void StateEnd()
    {
        
    }

    public virtual void StateUpdate()
    {
        
    }
    
    /// <summary>
    /// 进行球形映射用于解决斜边加速问题,手柄对斜边加速的问题进行了优化，所以手柄类无需调用此函数
    /// </summary>
    /// <param name="target"></param>
    /// <returns></returns>
    public Vector2 SquareToCircle(Vector2 target)
    {
        Vector2 output = Vector2.zero;
        output.x = target.x * Mathf.Sqrt(1 - (target.y * target.y) / 2.0f);
        output.y = target.y * Mathf.Sqrt(1 - (target.x * target.x) / 2.0f);
        return output;
    }
}

````

>State子类


* **手柄类**


````

using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class JoyStrickState : IInputState {

	[Header("===== Key Setting =====")]
	public string AxisX = "axisX"; //左摇杆X
	public string AxisY = "axisY"; //左摇杆Y

	public string LT = "LT"; //LT键
	public string RT = "RT"; //RT键
		
	public string Axis4 = "axis4"; //右摇杆X
	public string Axis5 = "axis5"; //右摇杆Y
	
	public string btn0 = "btn0"; //A
	public string btn1 = "btn1"; //B
	
	public string btn4 = "btn4"; //LB
	public string btn5 = "btn5"; //RB
	
	public string btn8 = "btn8"; //左摇杆按压(L3)
	public string btn9 = "btn9"; //右摇杆按压(R3)

	private float value = 0;
	
	//想继续定制某按钮的功能在这里实例化
	#region MyButton实例化
	
	public MyButton buttonA = new MyButton();
	public MyButton buttonB = new MyButton();
	 
	public MyButton buttonRB = new MyButton();
	public MyButton buttonRT = new MyButton();
	public MyButton buttonLB = new MyButton();
	public MyButton buttonLT = new MyButton();
	
	public MyButton buttonL3 = new MyButton();
	public MyButton buttonR3 = new MyButton();	
	

	#endregion

	public JoyStrickState(InputStateController mController, Transform trans) : base(mController, trans)
	{
		this.MStateName = "JoyStrickState";
	}

	public override void StateUpdate()
	{
		//接收按钮信号，true or false
		#region buttonX.GetSignal() 

		buttonA.GetSignal(Input.GetButton(btn0));
		buttonB.GetSignal(Input.GetButton(btn1));

		buttonL3.GetSignal(Input.GetButton(btn8));
		buttonR3.GetSignal(Input.GetButton(btn9));

		buttonLB.GetSignal(Input.GetButton(btn4));
		buttonRB.GetSignal(Input.GetButton(btn5));

		buttonRT.GetSignal(true, Input.GetAxis(RT));
		buttonLT.GetSignal(true, Input.GetAxis(LT));

		#endregion

		//---------------------------------------------------

		//左摇杆Y轴分量
		Target_Vertical = Input.GetAxis(AxisY);
		//左摇杆X轴分量
		Target_Horizontal = Input.GetAxis(AxisX);
		//右摇杆Y轴分量
		Jup = Input.GetAxis(Axis5);
		//右摇杆X轴分量
		Jright = -Input.GetAxis(Axis4);

		//是否禁用移动功能，移动分量归0
		if (!IsInputEnable)
		{
			Target_Horizontal = 0;
			Target_Vertical = 0;
		}

		/* 以下注释的为原来的代码。手柄模式下无需进行斜边值的映射，代码修改。
		Vector2 tempDAxis = SquareToCircle(new Vector2(Target_Horizontal, Target_Vertical));
		将平移值开方简化程序
		DMag = Mathf.Sqrt((tempDAxis.x * tempDAxis.x) + (tempDAxis.y * tempDAxis.y));
		*/

		//将控制移动的分量X,Y开方得到DMag
		DMag = Mathf.Sqrt((Target_Horizontal * Target_Horizontal) + (Target_Vertical * Target_Vertical));

		//计算转向,注意该脚本是挂载在角色控制柄上而非角色模型本身，旋转的是角色控制柄（角色模型是控制柄的子对象）
		Dvec = (Target_Horizontal * m_transform.right) + (Target_Vertical * m_transform.forward);

		//---------------------------------------------------
	
		//按钮信号的进一步判断，比如是按下，还是按住。
		#region 
        isrun = (buttonL3.isPressing);
		islock = buttonR3.OnPressed;
		isdefense = buttonLB.isPressing;
		isjump = buttonA.OnPressed;
		isFontStab = buttonB.OnPressed;

		rb = buttonRB.OnPressed;
		lb = buttonLB.OnPressed;
		rt = buttonRT.OnPressed;
		lt = buttonLT.OnPressed;
        #endregion

		value = CheckController.GetCurController();
		if (value == 1)
		{
			m_Controller.SetState(new KeyBoardState(m_Controller, m_transform));
		}

	}
}

````

* **键鼠类**

````
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class KeyBoardState : IInputState {

	public KeyBoardState(InputStateController mController, Transform trans) : base(mController, trans)
	{
		this.MStateName = "KeyBoardState";
	}

	[Header("===== Key Setting =====")]
	public string KeyUp = "w";
	public string KeyDown = "s";
	public string KeyLeft = "a";
	public string KeyRight = "d";

	public string KeyA = "left shift";
	public string KeyB = "space";
	public string KeyC;
	public string KeyD;
    
	public string KeyUpArrow = "up";
	public string KeyDownArrow = "down";
	public string KeyLeftArrow = "left";
	public string KeyRightArrow = "right";
	
	public bool isOpenMouse;
	public float mouseSpeedX;
	public float mouseSpeedY;
    private int value = 0;
	
	public override void StateUpdate()
	{
		Target_Vertical = (Input.GetKey(KeyUp) ? 1.0f : 0.0f) - (Input.GetKey(KeyDown) ? 1.0f : 0.0f);
		Target_Horizontal = (Input.GetKey(KeyRight) ? 1.0f : 0.0f) - (Input.GetKey(KeyLeft) ? 1.0f : 0.0f);
		if (!isOpenMouse)
		{
			Jup = (Input.GetKey(KeyUpArrow) ? 1.0f : 0.0f) - (Input.GetKey(KeyDownArrow) ? 1.0f : 0.0f);
			Jright = (Input.GetKey(KeyRightArrow) ? 1.0f : 0.0f) - (Input.GetKey(KeyLeftArrow) ? 1.0f : 0.0f);
		}
		else
		{
			Jup = Input.GetAxis("Mouse Y") * mouseSpeedX;
			Jright = Input.GetAxis("Mouse X") * mouseSpeedY;
		}

		if (!IsInputEnable)
		{
			Target_Horizontal = 0;
			Target_Vertical = 0;
		}

		//对键值的增长进行平滑化处理
		Current_Vertical = Mathf.SmoothDamp(Current_Vertical, Target_Vertical, ref Velocity_Vertical, 0.1f);
		Current_Horizontal = Mathf.SmoothDamp(Current_Horizontal, Target_Horizontal, ref Velocity_Horizontal, 0.1f);

		Vector2 tempDAxis = SquareToCircle(new Vector2(Current_Horizontal, Current_Vertical));

		//将平移值开方简化程序
		DMag = Mathf.Sqrt((tempDAxis.x * tempDAxis.x) + (tempDAxis.y * tempDAxis.y));
        
		//计算转向
		Dvec = (tempDAxis.x * m_transform.right) + (tempDAxis.y * m_transform.forward);

		
		isrun = Input.GetKey(KeyA);
		//isdefense = Input.GetKey(KeyD);
        
		//是否按下跳跃键
		bool newjump = Input.GetKey(KeyB);
		if (newjump != isjumplast && newjump == true)
		{
			isjump = true;
		}
		else
		{
			isjump = false;
		}
		isjumplast = newjump;
		
        
//		bool newattack = Input.GetKey(KeyC);
//		if (newattack != isattacklast && newattack == true)
//		{
//			rb = true;
//		}
//		else
//		{
//			rb = false;
//		}
//		isattacklast = newattack;
		
		value = CheckController.GetCurController();
		if (value == -1)
		{
			m_Controller.SetState(new JoyStrickState(m_Controller, m_transform));
		}
	}
}

````

>状态的控制中心

````
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class InputStateController : MonoBehaviour
{

	public IInputState m_state = null;
	private bool m_bRunbegin = false;
	private InputStateTest _test;

	private void Awake()
	{
		_test = GetComponent<InputStateTest>();
		SetState(new JoyStrickState(this, this.transform));
		_test.InputState = m_state;
	}

	private void Update()
	{
		StateUpdate();
		if (m_state != _test.InputState)
		{
			_test.InputState = m_state;
		}
	}

	public void SetState(IInputState state)
	{
		m_bRunbegin = false;
		
		if (m_state != null)
		{
			m_state.StateEnd();
		}

		m_state = state;
	}

	public void StateUpdate()
	{
		if (m_state != null && m_bRunbegin == false)
		{
			m_state.StateBegin();
			m_bRunbegin = true;
		}

		if (m_state != null)
		{
			m_state.StateUpdate();
		}
	}

	

}

````

>Test类

无论按下手柄的按键还是键盘的，控制台都会打印出"jump".

````

using System.Collections;
using System.Collections.Generic;
using System.Runtime.Serialization.Formatters;
using UnityEngine;

public class InputStateTest : MonoBehaviour
{

	private IInputState _inputState;
    public IInputState InputState
	{
		set { _inputState = value; }
		get { return _inputState; }
	}

	private void Start()
	{
	}

	private void Update()
	{
		if (_inputState.isjump)
		{
			Debug.Log("jump");
		}
	}
}

````

---

**以上代码只是部分，完整代码后续上传github**





