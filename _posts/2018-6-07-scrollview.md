---
layout:     post
title:      "UGUI实现页面滚动视图"
subtitle:   "UGUI相关"
date:       2018-06-07 12:00:00
author:     "LYQ"
header-img: "img/in-post/default-bg.jpg"
tags:
    - Unity
    - 读书笔记
    - UGUI
---

本文记录了页面滚动处理的实例代码，以便日后查阅。

 

## 写在前面

采用Scroll Rect组件的滚动视图，是一种UI，可以借助该组件来切换页面。需要用到ScrollRect , GridLayoutGroup以及拖拽事件。


 
----
## 页面处理代码的实现
 
* ViewController.cs
该类作为其他UI视图基本类的使用

````
using System.Collections;
using System.Collections.Generic;
using UnityEngine;

public class ViewController : MonoBehaviour {

	private RectTransform cachedRectTransform;

	public RectTransform CachedRectTransform
	{
		get
		{
			if(cachedRectTransform == null)
			    cachedRectTransform = GetComponent<RectTransform>();
			return cachedRectTransform;			
        }
	}
}
````
GetComponent的处理是比较缓慢的，频繁调用的话会导致负荷加大，影响性能。在第一次调用的时候进行缓存，第二次开始就可以直接调用。

* PagingScollViewController.cs
   该类执行页面处理的过程

````
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using UnityEngine.EventSystems;

public class PagingScorllViewControll : ViewController, 
IBeginDragHandler,IEndDragHandler
{
     private Vector2 initPosition;
	 private Vector2 destPosition;
     private bool isAnimation = false;
	 private AnimationCurve animationCurve;
     private int keepIndexNumber;
     private ScrollRect cachedScrollRect;

	 private Rect currentRect;

	 [SerializeField]private PageControl pageControl;
	 public ScrollRect CachedScrollRect
	 {
          get
		  {
			  if(cachedScrollRect == null)
			      cachedScrollRect = GetComponent<ScrollRect>();
			  return cachedScrollRect;
		  }
	 }

	 public void OnBeginDrag(PointerEventData evetdates)
	 {
         isAnimation = false;
	 }

	 public void OnEndDrag(PointerEventData evetdates)
	 {
		 GridLayoutGroup grid = cachedScrollRect.content.GetComponent
		 <GridLayoutGroup>();

		 CachedScrollRect.StopMovement();

		 float pageWidth = -(grid.cellSize.x + grid.spacing.x);

		 int pageIndex = Mathf.RoundToInt
		 (CachedScrollRect.content.anchoredPosition.x / pageWidth);

		 if(pageIndex == keepIndexNumber && Mathf.Abs(evetdates.delta.x)>=4)
		 {
              pageIndex += (int)Mathf.Sign(-evetdates.delta.x);
		 }

		 
		 if(pageIndex < 0)
		     pageIndex = 0;
		 else if(pageIndex > grid.transform.childCount - 1)
		     pageIndex = grid.transform.childCount - 1;
	     
		 float dest = pageIndex * pageWidth;

		 destPosition = new Vector2
		 (dest,CachedScrollRect.content.anchoredPosition.y);

		 initPosition = CachedScrollRect.content.anchoredPosition;
          
         Keyframe k1 = new Keyframe(Time.time,0,0,0);
		 Keyframe k2 = new Keyframe(Time.time+0.3f,1,0,0);
		 animationCurve = new AnimationCurve(k1,k2);

		 

		 isAnimation = true;
         pageControl.SetCurrentPage(pageIndex);


	 }

	 private void Start() {
		 UpdateView();
		 pageControl.SetNumberOfPages(5);
		 pageControl.SetCurrentPage(0);
	 }

	 private void Update() {
		 
		 if(CachedRectTransform.rect.width != currentRect.width 
		 || CachedRectTransform.rect.height != currentRect.height)
		 {
			 UpdateView();
		 }

		

	 }


     private void LateUpdate() {
	     if(isAnimation)
		 {
			 if(Time.time >= animationCurve.keys[animationCurve.length-1].time)
			 {
                  CachedScrollRect.content.anchoredPosition = destPosition;
				  isAnimation = false;
				  return;
			 }

			 Vector2 newPos = initPosition + (destPosition - initPosition)*
			 animationCurve.Evaluate(Time.time);

			 CachedScrollRect.content.anchoredPosition = newPos;
		 }	 
	 }
        //自动适应屏幕大小，使当前视图居中
	 public void UpdateView()
	 {
		 GridLayoutGroup grid = CachedScrollRect.content.GetComponent
		 <GridLayoutGroup>();
         
		 currentRect = CachedRectTransform.rect;

		 int PadingH = Mathf.RoundToInt
		 ((currentRect.width - grid.cellSize.x) /2 );

		 int Padingv = Mathf.RoundToInt
		 ((currentRect.height - grid.cellSize.y) /2 );  

		 grid.padding = new RectOffset(PadingH,PadingH,Padingv,Padingv);


	 }
	 
}

````
 ##### 上诉实现过程的关键就是，当超过一定速度拖拽的时候，就前进一页，实现流畅切换。

## 页面指示器代码相关

````
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

public class PageControl : MonoBehaviour {

	 [SerializeField]private Toggle indicator;
	 private List<Toggle> indicators = new List<Toggle>();

	 private void Awake() {
		 indicator.gameObject.SetActive(false);
	 }


     public void SetCurrentPage(int index)
	 {
		 if(index >= 0 && index <= indicators.Count-1)
		 {
			 indicators[index].isOn = true;
		 }
	 }
	 public void SetNumberOfPages(int number)
	 {
          if(indicators.Count<number)
		  {
			  for(int i = indicators.Count;i<number;i++)
			  {
				  Toggle tog = Instantiate(indicator) as Toggle;
				  tog.gameObject.SetActive(true);
				  tog.transform.SetParent(indicator.transform.parent);
				  tog.isOn = false;
                  indicators.Add(tog);
			  }
		  }

		  else if(indicators.Count>number)
		  {
			  for(int i = indicators.Count-1;i>=number;i--)
			  {
                   Destroy(indicators[i].gameObject);
				   indicators.RemoveAt(i);
			  }
		  }
	 }
}

````