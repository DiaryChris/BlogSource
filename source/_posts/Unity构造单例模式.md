---
title: Unity构造单例模式
date: 2019-04-09 17:59:42
tags:
---





```C#
public class GameManager : MonoBehaviour
{
	public static GameManager instance;

    private void Awake()
    {
        //singleton pattern
        if (!instance)
        {
            instance = this;
        }
        else if (instance != this)
        {
            DestroyImmediate(this);
        }
    }
}
```

