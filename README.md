# firebase-remote-config
Google Firebase (Firebase Remote Config)
## How To Install

### Add the lines below to `Packages/manifest.json`

for version `10.7.0`
```csharp
"com.google.firebase.remote-config": "https://github.com/google-for-unity/firebase-remote-config.git#10.7.0",
```

dependency `external-dependency-manager-1.2.175`, `firebase-app-core-10.7.0`
```csharp
"com.google.firebase.core": "https://github.com/google-for-unity/firebase-app-core.git#10.7.0",
"com.google.external-dependency-manager": "https://github.com/google-for-unity/external-dependency-manager-for-unity.git#1.2.175",
```

## Maybe you need
<details><summary>Template code here</summary>

```
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using Firebase;
using Firebase.Extensions;
using Firebase.RemoteConfig;
using UnityEngine;

public class FirebaseRemoteConfigManager
{
    static DependencyStatus dependencyStatus = DependencyStatus.UnavailableOther;

    public static bool IsInitialized = false;

    private const string KeyTestString = "Test_String";
    private const string KeyTestInt = "Test_Int";
    private const string KeyTestFloat = "Test_Float";
    private const string KeyTestBool = "Test_Bool";
    public static string stringValue = "";
    public static int intvalue = 0;
    public static float floatValue = 1.0f;
    public static bool boolValue = false;

    //Init firebase
    public static void Initialize()
    {
        FirebaseApp.CheckAndFixDependenciesAsync().ContinueWithOnMainThread(task =>
        {
            dependencyStatus = task.Result;
            if (dependencyStatus == DependencyStatus.Available)
            {
                InitFirebase();
            }
            else
            {
                Debug.LogError("Could not resolve all Firebase dependencies: " + dependencyStatus);
            }
        });
    }

    static async void InitFirebase()
    {
        Dictionary<string, object> defaultValue = new Dictionary<string, object>()
        {
            { KeyTestString, "Test_String" },
            { KeyTestInt, 1 },
            { KeyTestFloat, 1.0f },
            { KeyTestBool, false }
        };
        await FirebaseRemoteConfig.DefaultInstance.SetDefaultsAsync(defaultValue).ContinueWithOnMainThread(task =>
        {
// [END set_defaults]
            Debug.Log("RemoteConfig configured and ready!");
        });
        await FetchDataAsync();
    }

    public static Task FetchDataAsync()
    {
        Debug.Log("Fetching data...");
        System.Threading.Tasks.Task fetchTask =
            Firebase.RemoteConfig.FirebaseRemoteConfig.DefaultInstance.FetchAsync(
                TimeSpan.Zero);
        return fetchTask.ContinueWithOnMainThread(tast =>
        {
            var info = Firebase.RemoteConfig.FirebaseRemoteConfig.DefaultInstance.Info;
            //SET NEW DATA FROM REMOTE CONFIG
            if (info.LastFetchStatus == LastFetchStatus.Success)
            {
                FirebaseRemoteConfig.DefaultInstance.ActivateAsync()
                    .ContinueWithOnMainThread(task =>
                    {
                        Debug.Log(String.Format("Remote data loaded and ready (last fetch time {0}).",
                            info.FetchTime));
                    });
                stringValue = FirebaseRemoteConfig.DefaultInstance.GetValue(KeyTestString).StringValue;
                intvalue = int.Parse(FirebaseRemoteConfig.DefaultInstance.GetValue(KeyTestInt).StringValue);
                floatValue = float.Parse(FirebaseRemoteConfig.DefaultInstance.GetValue(KeyTestFloat).StringValue);
                boolValue = FirebaseRemoteConfig.DefaultInstance.GetValue(KeyTestBool).BooleanValue;
            }
            else
            {
                Debug.Log("Fetching data did not completed!");
            }

            IsInitialized = true;
        });
    }
}

```
