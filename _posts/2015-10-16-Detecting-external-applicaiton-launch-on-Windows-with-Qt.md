---
layout: post
title: "How to detect an external appliction launch on Windows with Qt 5"
modified: 2015-10-15 00:00:00 -0700
tags: [Qt,Qt5,Shellhook,nativeEventFilter,Windows,Shell,ShellHookWindow,Hook,HSHELL_WINDOWCREATED,media detection,RegisterShellHookWindow]
image:
  feature: cplusplus.jpg
comments: 
share: 
---

## Simple Media Player Launch detection

In a project of mine,I required to detect certain application launch(media players,MPC for example) events on my app. It turned out,on Windows this is pretty easy.This tutorial assumes you already know basic stuff,
so if you end up here through Google this will be a nice guide.

## Step 1

First we need to know about Windows Events.If it would be a native app,we'd have a WndProc.On Qt, we have QAbstractNativeEventFilter.You wrap this class and give to the QCoreApplication


{% highlight ruby pygments %}

//Main.cpp
YourQCoreApplicationInstance::installNativeEventFilter(QAbstractNativeEventFilter*)
//~ 



class AbstractNativeEventFilterHelper : public QAbstractNativeEventFilter
{
..
	virtual bool nativeEventFilter(const QByteArray &eventType,void *message,long *) Q_DECL_OVERRIDE;
..
}

{% endhighlight %}

Then your class will receive all windows events that Qt event dispatcher implements.

## Step 2

Register the main window HWND. For the moment,I dont know a nice way to get the main window HWND so I implemented a work around.I get the hwnd on WM_CREATED message.

{% highlight ruby pygments %}

if(msg->message == WM_CREATE)
{
	m_pHwnd = msg->hwnd;
	..
}
{% endhighlight %}

## Step 3

Register SHELLHOOK message.

{% highlight ruby pygments %}
if(msg->message == WM_CREATE)
{
	m_pHwnd = msg->hwnd;
	if(m_pHwnd)
	{
		RegisterWindowMessage(TEXT("SHELLHOOK"));
		if(RegisterShellHookWindow(m_pHwnd))
		{
			qDebug() << "ShellHook registered";
			GetWindowThreadProcessId(m_pHwnd,&m_pId);
		}
	}
}
{% endhighlight %}

Note that I didnt test this code against complicated cases. So there might be cases where when you receive multiple WM_CREATEs.

## Step 4

Receive shell messages!
{% highlight ruby pygments %}
if(msg->wParam == HSHELL_WINDOWCREATED)
{
	DWORD handle;
	GetWindowThreadProcessId((HWND)msg->lParam,&handle);

	
	m_pDetectedWindow = (HWND)msg->lParam;


	//...Do whatever you like with the window..
	//Here is how to get the launched window title
	//
	//char className[256];
	//GetClassNameA(m_pDetectedWindow,className,256);

	//char windowValue[256];
	//GetWindowTextA(m_pDetectedWindow,windowValue,256);
	
}
{% endhighlight %}

Look into HSHELL messages for more info.

## Full Code
{% highlight ruby pygments %}
void WindowsMediaDetection::OnEvent(const QByteArray &eventType,void *message,long *)
{
	if(eventType == "windows_generic_MSG" || eventType == "windows_dispatcher_MSG")
	{
		MSG* msg = (MSG*)message;

		if(msg->message == WM_CREATE)
		{
			m_pHwnd = msg->hwnd;
			if(m_pHwnd)
			{
				RegisterWindowMessage(TEXT("SHELLHOOK"));
				if(RegisterShellHookWindow(m_pHwnd))
				{
					qDebug() << "ShellHook registered";
					GetWindowThreadProcessId(m_pHwnd,&m_pId);
				}
			}
		}
		if(msg->wParam == HSHELL_WINDOWCREATED) //This will get called everytime a window created
		{
			DWORD handle;
			GetWindowThreadProcessId((HWND)msg->lParam,&handle);

		
			m_pDetectedWindow = (HWND)msg->lParam;
		}
	}
}
{% endhighlight %}


