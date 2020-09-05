##### important points：  

1. It is the context of the current state of the application.  
2. It can be used to get information regarding the activity and application.
3. It can be used to get access to resources, databases, and shared preferences, and etc.
4. Both the Activity and Application classes extend the Context class.  
  

##### Mainly two types of context
> Wrong use of Context can easily lead to memory leaks in an android application.  

1. Application Context: It is the application and we are present in Application. For example - MyApplication(which extends Application class). It is an instance of MyApplication only.  
```
It is an instance that is the singleton and can be accessed in activity via getApplicationContext().
This context is tied to the lifecycle of an application. 
The application context can be used where you need a context whose lifecycle is separate from 
the current context or when you are passing a context beyond the scope of activity.


Example Use: If you have to create a singleton object for your application and that object needs a context, 
always pass the application context.

If you pass the activity context here, it will lead to the memory leak as it will keep the 
reference to the activity and activity will not be garbage collected.

In case, when you have to initialize a library in an activity, always pass the 
application context, not the activity context.

You only use getApplicationContext() when you know you need a 
Context for something that may live longer than any other likely 
Context you have at your disposal.
```
2. Activity Context: It is the activity and we are present in Activity. For example - MainActivity. It is an instance of MainActivity only.    

```
This context is available in an activity. This context is tied to the lifecycle of an activity. 
The activity context should be used when you are passing the context in the scope of an activity
or you need the context whose lifecycle is attached to the current context.

Example Use: If you have to create an object whose lifecycle is attached to an activity,
 you can use the activity context.
```

##### ContentProvider

This context is the application context and can be used as the application context. 
You can get it using the getContext() method.


##### When to use which Context?  
always remember, in case of Singleton(lifecycle is attached to the application lifecycle), 
always use the Application Context.

Whenever you are in Activity, for any UI operations like showing toast, dialogs, 
and etc, use the Activity Context.

Always try to use the nearest context which is available to you. When you are in Activity, the nearest context is Activity context. When you are in Application, the nearest context is the Application context. If Singleton, use the Application Context.

##### 参考链接  
```
https://blog.mindorks.com/understanding-context-in-android-application-330913e32514
```


